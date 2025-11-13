# PDF Section Extraction Architecture V3 - Redesign
**Date:** November 13, 2025
**Status:** Proposed Solution
**Objective:** Achieve 95%+ coverage with ±15% section count accuracy

---

## Executive Summary

### The Core Insight
**Current architecture fails because it confuses two distinct problems:**
1. **Block Detection** - "Where are the visual blocks in the document?"
2. **Block Decomposition** - "How should each block be split into sections?"

Current approach tries to solve both simultaneously with gap thresholds → FAILS.

**New approach: Separate these concerns across multiple passes.**

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│ PASS 1: Document Analysis & Signature Learning         │
│  - Extract all text items from PDF.js                  │
│  - Build statistical signature (gaps, fonts, margins)  │
│  - Detect columns, overall layout                      │
│  Output: Document signature + sorted text items        │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│ PASS 2: Visual Block Detection (Whitespace Clustering) │
│  - Cluster text items into visual blocks using gaps    │
│  - Use percentile-based thresholds, not fixed values   │
│  - Detect natural whitespace boundaries                │
│  Output: Array of TextBlock objects                    │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│ PASS 3: Block Classification                           │
│  - Analyze internal structure of each block            │
│  - Score against block types (paragraph, list, header) │
│  - Use multi-signal scoring, not brittle regex         │
│  Output: Typed blocks with confidence scores           │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│ PASS 4: Block Decomposition (Type-Specific Rules)      │
│  - Apply type-specific decomposition rules             │
│  - Paragraph → one section                             │
│  - List → split by bullet markers                      │
│  - Table → keep whole or split by rows                 │
│  Output: Array of Section objects                      │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│ PASS 5: Reading Order & Validation                     │
│  - Sort sections by visual position (column-aware)     │
│  - Assign P1, P2, P3... in reading order               │
│  - Validate coverage, detect overlaps                  │
│  - Quality checks and confidence scoring               │
│  Output: Final numbered sections with metadata         │
└─────────────────────────────────────────────────────────┘
```

---

## Pass 1: Document Analysis & Signature Learning

### Objectives
- Extract raw text items from PDF.js
- Build statistical profile of document layout
- Detect multi-column layout if present
- Sort items in rough reading order

### Algorithm

```javascript
class DocumentAnalyzer {
    async analyze(page) {
        // 1. Extract text items from PDF.js
        const textContent = await page.getTextContent();
        const items = this.transformCoordinates(textContent.items);

        // 2. Sort by Y position (top to bottom)
        items.sort((a, b) => a.y - b.y);

        // 3. Build statistical signature
        const signature = this.buildSignature(items);

        // 4. Detect columns
        const columnInfo = this.detectColumns(items, signature);

        // 5. Sort items by reading order (column-aware)
        const sortedItems = this.sortByReadingOrder(items, columnInfo);

        return {
            items: sortedItems,
            signature: signature,
            columnInfo: columnInfo
        };
    }

    buildSignature(items) {
        // Collect all vertical gaps
        const gaps = [];
        for (let i = 0; i < items.length - 1; i++) {
            const gap = items[i + 1].y - (items[i].y + items[i].height);
            if (gap >= 0 && gap < 100) { // Filter outliers
                gaps.push(gap);
            }
        }

        // Calculate gap percentiles (more robust than k-means)
        gaps.sort((a, b) => a - b);
        const p25 = this.percentile(gaps, 25);
        const p50 = this.percentile(gaps, 50);
        const p75 = this.percentile(gaps, 75);
        const p90 = this.percentile(gaps, 90);

        // Collect font sizes
        const fontSizes = items.map(item => item.fontSize);
        const dominantFontSize = this.mode(fontSizes);

        // Collect X positions (left margins)
        const xPositions = items.map(item => item.x);
        const commonMargins = this.clusterValues(xPositions, tolerance = 10);

        return {
            gapPercentiles: { p25, p50, p75, p90 },
            dominantFontSize: dominantFontSize,
            commonLeftMargins: commonMargins,
            pageWidth: items[0]?.pageWidth || 850,
            pageHeight: items[0]?.pageHeight || 1100,
            totalItems: items.length
        };
    }

    percentile(sortedArray, p) {
        const index = Math.floor(sortedArray.length * p / 100);
        return sortedArray[index];
    }

    mode(array) {
        const frequency = {};
        let maxFreq = 0;
        let mode = array[0];

        array.forEach(val => {
            frequency[val] = (frequency[val] || 0) + 1;
            if (frequency[val] > maxFreq) {
                maxFreq = frequency[val];
                mode = val;
            }
        });

        return mode;
    }
}
```

### Key Improvements Over V2
1. **Percentile-based gaps** instead of k-means (more robust to outliers)
2. **Multiple gap thresholds** (p25, p50, p75, p90) for different use cases
3. **Statistical mode** for dominant font size (more robust than mean)
4. **Outlier filtering** in gap calculation (exclude page breaks, etc.)

---

## Pass 2: Visual Block Detection (Whitespace Clustering)

### Core Insight
**Don't use fixed thresholds. Use statistical clustering to detect natural boundaries.**

The key observation: Large gaps separate blocks, small gaps connect items within blocks.

### Algorithm: Adaptive Gap-Based Clustering

```javascript
class VisualBlockDetector {
    detectBlocks(items, signature) {
        const blocks = [];
        let currentBlock = {
            items: [items[0]],
            startY: items[0].y,
            endY: items[0].y + items[0].height
        };

        // Dynamic threshold: use 75th percentile as block boundary
        // Items separated by gap > p75 are in different blocks
        const blockBoundaryThreshold = signature.gapPercentiles.p75;

        for (let i = 1; i < items.length; i++) {
            const prevItem = items[i - 1];
            const currentItem = items[i];

            // Calculate gap between items
            const gap = currentItem.y - (prevItem.y + prevItem.height);

            // Decision: Is this a block boundary?
            const isBlockBoundary = this.isBlockBoundary(
                gap,
                currentItem,
                prevItem,
                blockBoundaryThreshold,
                signature
            );

            if (isBlockBoundary) {
                // Finalize current block
                blocks.push(this.finalizeBlock(currentBlock));

                // Start new block
                currentBlock = {
                    items: [currentItem],
                    startY: currentItem.y,
                    endY: currentItem.y + currentItem.height
                };
            } else {
                // Add to current block
                currentBlock.items.push(currentItem);
                currentBlock.endY = currentItem.y + currentItem.height;
            }
        }

        // Finalize last block
        if (currentBlock.items.length > 0) {
            blocks.push(this.finalizeBlock(currentBlock));
        }

        return blocks;
    }

    isBlockBoundary(gap, currentItem, prevItem, threshold, signature) {
        // PRIMARY SIGNAL: Gap significantly larger than typical
        if (gap >= threshold) {
            return true;
        }

        // SECONDARY SIGNALS: Even with small gap, break if:

        // 1. Font size changed significantly (header vs body)
        const fontChange = Math.abs(currentItem.fontSize - prevItem.fontSize);
        if (fontChange > signature.dominantFontSize * 0.2) {
            return true;
        }

        // 2. Large horizontal position change (column switch, indent change)
        const xChange = Math.abs(currentItem.x - prevItem.x);
        if (xChange > 50) {
            return true;
        }

        // 3. Text pattern indicates new section (numbered heading)
        if (this.looksLikeHeader(currentItem) && !this.looksLikeHeader(prevItem)) {
            return true;
        }

        return false;
    }

    looksLikeHeader(item) {
        const text = item.text.trim();

        // Numbered heading: "1. Title", "2.1 Subtitle"
        if (/^\d+\.(\d+\.)?\s/.test(text)) return true;

        // All caps short text
        if (text === text.toUpperCase() && text.length > 3 && text.length < 50) return true;

        // Large font
        if (item.fontSize > item.dominantFontSize * 1.15) return true;

        return false;
    }

    finalizeBlock(block) {
        return {
            items: block.items,
            text: block.items.map(item => item.text).join(' '),
            boundingBox: {
                x: Math.min(...block.items.map(item => item.x)),
                y: block.startY,
                width: Math.max(...block.items.map(item => item.x + item.width)) -
                       Math.min(...block.items.map(item => item.x)),
                height: block.endY - block.startY
            },
            itemCount: block.items.length
        };
    }
}
```

### Why This Works Better

**Problem with V2:**
```
Document A: gaps = [12, 12, 15, 12, 15, 12]
p75 = 15, threshold = 15 * 0.85 = 12.75
Result: Breaks on gaps of 15px (correct) but also 12px (WRONG - too sensitive)

Document B: gaps = [14, 14, 28, 14, 28, 14]
p75 = 28, threshold = 28 * 0.85 = 23.8
Result: Breaks on gaps of 28px but misses gaps of 20px (WRONG - misses breaks)
```

**Solution in V3:**
```
Document A: gaps = [12, 12, 15, 12, 15, 12]
p75 = 15, threshold = 15 (no multiplier!)
Result: Breaks on gaps >= 15px, continues on gaps of 12px (CORRECT)

Document B: gaps = [14, 14, 28, 14, 28, 14]
p75 = 28, threshold = 28
Result: Breaks on gaps >= 28px, continues on gaps of 14px (CORRECT)

Document C: gaps = [10, 11, 12, 22, 23, 24]
p75 = 23, threshold = 23
Result: Breaks on gaps >= 23px, continues on gaps of 10-12px (CORRECT)
```

**Key insight: The 75th percentile IS the threshold. Don't multiply it by magic numbers.**

---

## Pass 3: Block Classification

### Objectives
- Determine the type of each visual block
- Use multi-signal scoring (not brittle regex)
- Assign confidence scores

### Block Types
1. **section-header** - Standalone heading (numbered, large font, all caps)
2. **body-paragraph** - Regular text block
3. **bullet-list** - Multiple bullet items
4. **table** - Structured data
5. **numbered-list** - Ordered list (1., 2., 3.)

### Algorithm: Multi-Signal Scoring

```javascript
class BlockClassifier {
    classify(block, signature) {
        const scores = {
            'section-header': this.scoreAsHeader(block, signature),
            'body-paragraph': this.scoreAsParagraph(block, signature),
            'bullet-list': this.scoreAsBulletList(block, signature),
            'table': this.scoreAsTable(block, signature),
            'numbered-list': this.scoreAsNumberedList(block, signature)
        };

        // Find highest scoring type
        let bestType = 'body-paragraph'; // default
        let bestScore = scores['body-paragraph'];

        for (const [type, score] of Object.entries(scores)) {
            if (score > bestScore) {
                bestScore = score;
                bestType = type;
            }
        }

        return {
            type: bestType,
            confidence: bestScore,
            allScores: scores
        };
    }

    scoreAsHeader(block, signature) {
        let score = 0;
        const text = block.text.trim();
        const firstItem = block.items[0];

        // Signal 1: Numbered heading pattern (strong signal)
        if (/^\d+\.(\d+\.)?\s+[A-Z]/.test(text)) {
            score += 5;
        }

        // Signal 2: All caps (medium signal)
        if (text === text.toUpperCase() && text.length > 3 && text.length < 80) {
            score += 3;
        }

        // Signal 3: Larger font than dominant (strong signal)
        if (firstItem.fontSize > signature.dominantFontSize * 1.1) {
            score += 4;
        }

        // Signal 4: Short text (weak signal - headers are usually concise)
        const wordCount = text.split(/\s+/).length;
        if (wordCount <= 10) {
            score += 1;
        }

        // Signal 5: Single line (weak signal)
        if (block.items.length === 1) {
            score += 1;
        }

        // Anti-signal: Too long for header
        if (wordCount > 30) {
            score -= 3;
        }

        return score;
    }

    scoreAsParagraph(block, signature) {
        let score = 3; // Base score (default type)
        const text = block.text.trim();
        const wordCount = text.split(/\s+/).length;

        // Signal 1: Moderate length (10-200 words)
        if (wordCount >= 10 && wordCount <= 200) {
            score += 2;
        }

        // Signal 2: Multiple lines
        if (block.items.length >= 2) {
            score += 1;
        }

        // Signal 3: Normal font size
        const avgFontSize = block.items.reduce((sum, item) => sum + item.fontSize, 0) / block.items.length;
        if (Math.abs(avgFontSize - signature.dominantFontSize) < 1) {
            score += 1;
        }

        // Signal 4: Starts at common left margin
        if (this.isAtCommonMargin(block.items[0].x, signature.commonLeftMargins)) {
            score += 1;
        }

        return score;
    }

    scoreAsBulletList(block, signature) {
        let score = 0;

        // Count items starting with bullet markers
        const bulletPattern = /^[\u2022\u25CF\u25E6\u25AA\u25AB\u2023\u2043\u204C\u204D\u2219\u29BE\u29BF]/;
        const bulletCount = block.items.filter(item => bulletPattern.test(item.text.trim())).length;

        // Signal 1: Multiple bullet markers (very strong signal)
        if (bulletCount >= 2) {
            score += 10;
        } else if (bulletCount === 1) {
            score += 3;
        }

        // Signal 2: Consistent indentation
        const xPositions = block.items.map(item => item.x);
        const xVariance = this.variance(xPositions);
        if (xVariance < 10) { // Low variance = consistent indent
            score += 2;
        }

        return score;
    }

    scoreAsTable(block, signature) {
        let score = 0;
        const text = block.text;

        // Signal 1: Multiple pipe characters (|) in multiple lines
        const linesWithPipes = block.items.filter(item => item.text.includes('|')).length;
        if (linesWithPipes >= 2) {
            score += 5;
        }

        // Signal 2: Consistent column structure (tab-separated or aligned)
        const hasConsistentColumns = this.detectColumnStructure(block.items);
        if (hasConsistentColumns) {
            score += 4;
        }

        // Anti-signal: Just contact info with pipe (false positive)
        if (block.items.length === 1 && linesWithPipes === 1) {
            score -= 3;
        }

        return score;
    }

    scoreAsNumberedList(block, signature) {
        let score = 0;

        // Count items starting with numbers
        const numberedPattern = /^(\d+\.|\d+\))\s/;
        const numberedCount = block.items.filter(item => numberedPattern.test(item.text.trim())).length;

        // Signal 1: Multiple numbered items
        if (numberedCount >= 2) {
            score += 8;
        } else if (numberedCount === 1) {
            // Could be a header, not a list
            score += 2;
        }

        // Signal 2: Sequential numbering
        const numbers = block.items
            .map(item => {
                const match = item.text.match(/^(\d+)/);
                return match ? parseInt(match[1]) : null;
            })
            .filter(n => n !== null);

        if (this.isSequential(numbers)) {
            score += 3;
        }

        return score;
    }

    isSequential(numbers) {
        if (numbers.length < 2) return false;
        for (let i = 1; i < numbers.length; i++) {
            if (numbers[i] !== numbers[i-1] + 1) return false;
        }
        return true;
    }

    isAtCommonMargin(x, commonMargins) {
        return commonMargins.some(margin => Math.abs(x - margin) < 10);
    }

    variance(values) {
        const mean = values.reduce((sum, val) => sum + val, 0) / values.length;
        const squaredDiffs = values.map(val => Math.pow(val - mean, 2));
        return squaredDiffs.reduce((sum, val) => sum + val, 0) / values.length;
    }

    detectColumnStructure(items) {
        // Simple heuristic: Check if items have similar X positions at regular intervals
        // This is a placeholder - could be enhanced with more sophisticated logic
        return false;
    }
}
```

### Why Multi-Signal Scoring Works

**Problem with V2 (brittle regex):**
```javascript
// "1. Title" → Always header (WRONG if it's first item of numbered list)
// "NVIDIA" → Always header (WRONG if it's just a company name)
// "• bullet" → Always bullet (WRONG if bullet marker is just decoration)
```

**Solution in V3 (scoring):**
```javascript
Block: "1. Introduction to NVIDIA Architecture"

Header score:
  + 5 (numbered heading pattern: "1. ")
  + 0 (not all caps)
  + 0 (normal font size)
  + 1 (short - 5 words)
  = 6

Numbered list score:
  + 2 (one numbered item)
  + 0 (not sequential - only one item)
  = 2

Result: Classified as header (6 > 2) ✓

---

Block: "1. First item\n2. Second item\n3. Third item"

Header score:
  + 5 (numbered pattern)
  - 3 (too long - 30+ words)
  = 2

Numbered list score:
  + 8 (three numbered items)
  + 3 (sequential: 1, 2, 3)
  = 11

Result: Classified as numbered-list (11 > 2) ✓
```

---

## Pass 4: Block Decomposition (Type-Specific Rules)

### Objectives
- Split blocks into final sections
- Apply type-specific decomposition logic
- Handle bullet wrapping, numbered lists, etc.

### Algorithm

```javascript
class BlockDecomposer {
    decompose(typedBlock, signature) {
        switch (typedBlock.type) {
            case 'section-header':
                return this.decomposeHeader(typedBlock);

            case 'body-paragraph':
                return this.decomposeParagraph(typedBlock);

            case 'bullet-list':
                return this.decomposeBulletList(typedBlock);

            case 'numbered-list':
                return this.decomposeNumberedList(typedBlock);

            case 'table':
                return this.decomposeTable(typedBlock);

            default:
                return this.decomposeParagraph(typedBlock);
        }
    }

    decomposeHeader(block) {
        // Headers are always single sections
        return [{
            type: 'section-header',
            items: block.items,
            text: block.text,
            boundingBox: block.boundingBox
        }];
    }

    decomposeParagraph(block) {
        // Check if this is actually multiple paragraphs with small gaps
        const internalGaps = this.calculateInternalGaps(block.items);

        // If there are significant internal gaps, split into sub-blocks
        if (this.hasSignificantInternalGaps(internalGaps)) {
            return this.splitByInternalGaps(block, internalGaps);
        }

        // Otherwise, single paragraph
        return [{
            type: 'body-paragraph',
            items: block.items,
            text: block.text,
            boundingBox: block.boundingBox
        }];
    }

    decomposeBulletList(block) {
        const sections = [];
        let currentBullet = null;

        const bulletPattern = /^[\u2022\u25CF\u25E6\u25AA\u25AB\u2023\u2043\u204C\u204D\u2219\u29BE\u29BF]/;

        for (const item of block.items) {
            const text = item.text.trim();

            if (bulletPattern.test(text)) {
                // Start new bullet
                if (currentBullet) {
                    sections.push(this.finalizeBulletSection(currentBullet));
                }

                currentBullet = {
                    items: [item],
                    startY: item.y
                };
            } else if (currentBullet) {
                // Continuation of current bullet
                // Check if this is actually a continuation (indented, small gap)
                const prevItem = currentBullet.items[currentBullet.items.length - 1];
                const gap = item.y - (prevItem.y + prevItem.height);
                const xIndent = item.x - prevItem.x;

                // Heuristic: continuation if gap is small or item is indented
                if (gap < 20 || xIndent > 5) {
                    currentBullet.items.push(item);
                } else {
                    // Gap too large - this might be a new paragraph after the list
                    // Finalize bullet and treat this item as orphan (will be handled in validation)
                    sections.push(this.finalizeBulletSection(currentBullet));
                    currentBullet = null;

                    // Create standalone section for this orphan
                    sections.push({
                        type: 'body-paragraph',
                        items: [item],
                        text: item.text,
                        boundingBox: {
                            x: item.x,
                            y: item.y,
                            width: item.width,
                            height: item.height
                        }
                    });
                }
            } else {
                // Orphan item (text before first bullet - shouldn't happen if block detection worked)
                sections.push({
                    type: 'body-paragraph',
                    items: [item],
                    text: item.text,
                    boundingBox: {
                        x: item.x,
                        y: item.y,
                        width: item.width,
                        height: item.height
                    }
                });
            }
        }

        // Finalize last bullet
        if (currentBullet) {
            sections.push(this.finalizeBulletSection(currentBullet));
        }

        return sections;
    }

    decomposeNumberedList(block) {
        const sections = [];
        let currentItem = null;

        const numberedPattern = /^(\d+\.|\d+\))\s/;

        for (const item of block.items) {
            const text = item.text.trim();

            if (numberedPattern.test(text)) {
                // Start new numbered item
                if (currentItem) {
                    sections.push(this.finalizeNumberedSection(currentItem));
                }

                currentItem = {
                    items: [item],
                    startY: item.y
                };
            } else if (currentItem) {
                // Continuation of current numbered item
                currentItem.items.push(item);
            }
        }

        // Finalize last item
        if (currentItem) {
            sections.push(this.finalizeNumberedSection(currentItem));
        }

        return sections;
    }

    decomposeTable(block) {
        // Simple approach: Treat whole table as one section
        // Could be enhanced to split by rows if needed
        return [{
            type: 'table',
            items: block.items,
            text: block.text,
            boundingBox: block.boundingBox
        }];
    }

    finalizeBulletSection(bullet) {
        return {
            type: 'bullet-item',
            items: bullet.items,
            text: bullet.items.map(item => item.text).join(' '),
            boundingBox: {
                x: Math.min(...bullet.items.map(item => item.x)),
                y: bullet.items[0].y,
                width: Math.max(...bullet.items.map(item => item.x + item.width)) -
                       Math.min(...bullet.items.map(item => item.x)),
                height: bullet.items[bullet.items.length - 1].y +
                       bullet.items[bullet.items.length - 1].height - bullet.items[0].y
            }
        };
    }

    finalizeNumberedSection(item) {
        return {
            type: 'numbered-item',
            items: item.items,
            text: item.items.map(i => i.text).join(' '),
            boundingBox: {
                x: Math.min(...item.items.map(i => i.x)),
                y: item.items[0].y,
                width: Math.max(...item.items.map(i => i.x + i.width)) -
                       Math.min(...item.items.map(i => i.x)),
                height: item.items[item.items.length - 1].y +
                       item.items[item.items.length - 1].height - item.items[0].y
            }
        };
    }

    calculateInternalGaps(items) {
        const gaps = [];
        for (let i = 0; i < items.length - 1; i++) {
            gaps.push({
                index: i,
                gap: items[i + 1].y - (items[i].y + items[i].height)
            });
        }
        return gaps;
    }

    hasSignificantInternalGaps(gaps) {
        // Check if any gap is significantly larger than median
        const gapValues = gaps.map(g => g.gap);
        const median = this.median(gapValues);

        return gaps.some(g => g.gap > median * 2);
    }

    splitByInternalGaps(block, internalGaps) {
        // Split block at significant gaps
        const sections = [];
        let currentSection = { items: [block.items[0]] };

        for (let i = 0; i < internalGaps.length; i++) {
            const gap = internalGaps[i];

            if (this.isSignificantGap(gap, internalGaps)) {
                // Finalize current section
                sections.push({
                    type: 'body-paragraph',
                    items: currentSection.items,
                    text: currentSection.items.map(item => item.text).join(' '),
                    boundingBox: this.calculateBoundingBox(currentSection.items)
                });

                // Start new section
                currentSection = { items: [block.items[gap.index + 1]] };
            } else {
                // Continue current section
                currentSection.items.push(block.items[gap.index + 1]);
            }
        }

        // Finalize last section
        if (currentSection.items.length > 0) {
            sections.push({
                type: 'body-paragraph',
                items: currentSection.items,
                text: currentSection.items.map(item => item.text).join(' '),
                boundingBox: this.calculateBoundingBox(currentSection.items)
            });
        }

        return sections;
    }

    isSignificantGap(gap, allGaps) {
        const gapValues = allGaps.map(g => g.gap);
        const median = this.median(gapValues);
        return gap.gap > median * 2;
    }

    median(values) {
        const sorted = [...values].sort((a, b) => a - b);
        const mid = Math.floor(sorted.length / 2);
        return sorted.length % 2 === 0
            ? (sorted[mid - 1] + sorted[mid]) / 2
            : sorted[mid];
    }

    calculateBoundingBox(items) {
        return {
            x: Math.min(...items.map(item => item.x)),
            y: items[0].y,
            width: Math.max(...items.map(item => item.x + item.width)) -
                   Math.min(...items.map(item => item.x)),
            height: items[items.length - 1].y + items[items.length - 1].height - items[0].y
        };
    }
}
```

---

## Pass 5: Reading Order & Validation

### Objectives
- Sort sections by visual position (column-aware)
- Assign sequential paragraph numbers (P1, P2, P3...)
- Validate coverage and detect issues
- Calculate confidence scores

### Algorithm

```javascript
class ReadingOrderValidator {
    finalizeAndValidate(sections, allTextItems, columnInfo, signature) {
        // 1. Sort sections by reading order
        const sortedSections = this.sortByReadingOrder(sections, columnInfo);

        // 2. Assign paragraph numbers
        const numberedSections = sortedSections.map((section, index) => ({
            ...section,
            id: `p${index + 1}`,
            readingOrder: index + 1
        }));

        // 3. Validate coverage
        const validation = this.validateCoverage(numberedSections, allTextItems);

        // 4. Calculate confidence scores
        const scoredSections = numberedSections.map(section => ({
            ...section,
            confidence: this.calculateConfidence(section, validation)
        }));

        return {
            sections: scoredSections,
            validation: validation
        };
    }

    sortByReadingOrder(sections, columnInfo) {
        if (!columnInfo.isMultiColumn) {
            // Single column: sort by Y position
            return sections.sort((a, b) => a.boundingBox.y - b.boundingBox.y);
        } else {
            // Multi-column: sort left column first, then right column
            const leftColumnSections = sections
                .filter(s => s.boundingBox.x < columnInfo.boundary)
                .sort((a, b) => a.boundingBox.y - b.boundingBox.y);

            const rightColumnSections = sections
                .filter(s => s.boundingBox.x >= columnInfo.boundary)
                .sort((a, b) => a.boundingBox.y - b.boundingBox.y);

            return [...leftColumnSections, ...rightColumnSections];
        }
    }

    validateCoverage(sections, allTextItems) {
        // Track which text items are covered
        const coveredItems = new Set();

        sections.forEach(section => {
            section.items.forEach(item => {
                coveredItems.add(item.id); // Assuming items have unique IDs
            });
        });

        const totalItems = allTextItems.length;
        const coveredCount = coveredItems.size;
        const coveragePercent = (coveredCount / totalItems) * 100;

        // Find uncovered items
        const uncoveredItems = allTextItems.filter(item => !coveredItems.has(item.id));

        // Detect overlaps (items appearing in multiple sections)
        const itemCounts = new Map();
        sections.forEach(section => {
            section.items.forEach(item => {
                itemCounts.set(item.id, (itemCounts.get(item.id) || 0) + 1);
            });
        });

        const overlappingItems = Array.from(itemCounts.entries())
            .filter(([id, count]) => count > 1)
            .map(([id]) => allTextItems.find(item => item.id === id));

        return {
            totalItems: totalItems,
            coveredItems: coveredCount,
            coveragePercent: coveragePercent,
            uncoveredItems: uncoveredItems,
            overlappingItems: overlappingItems,
            isValid: coveragePercent >= 95 && overlappingItems.length === 0
        };
    }

    calculateConfidence(section, validation) {
        let confidence = 0.5; // Base confidence

        // Factor 1: Type confidence from classification
        if (section.typeConfidence) {
            confidence += section.typeConfidence * 0.3;
        }

        // Factor 2: Coverage quality
        confidence += (validation.coveragePercent / 100) * 0.2;

        // Factor 3: Item count (more items = more confident it's real section)
        if (section.items.length >= 2) {
            confidence += 0.1;
        }

        // Factor 4: Text length (very short sections might be noise)
        const wordCount = section.text.split(/\s+/).length;
        if (wordCount >= 5) {
            confidence += 0.1;
        }

        return Math.min(confidence, 1.0);
    }
}
```

---

## Complete Integration

### Main Orchestrator

```javascript
class StructuredPDFParserV3 {
    async parsePage(page) {
        // PASS 1: Document Analysis
        const analyzer = new DocumentAnalyzer();
        const { items, signature, columnInfo } = await analyzer.analyze(page);

        console.log('Document Signature:', signature);
        console.log('Total items:', items.length);

        // PASS 2: Visual Block Detection
        const blockDetector = new VisualBlockDetector();
        const blocks = blockDetector.detectBlocks(items, signature);

        console.log('Detected blocks:', blocks.length);

        // PASS 3: Block Classification
        const classifier = new BlockClassifier();
        const typedBlocks = blocks.map(block => ({
            ...block,
            ...classifier.classify(block, signature)
        }));

        console.log('Block types:',
            typedBlocks.map(b => `${b.type} (${b.confidence.toFixed(2)})`));

        // PASS 4: Block Decomposition
        const decomposer = new BlockDecomposer();
        const allSections = typedBlocks.flatMap(block =>
            decomposer.decompose(block, signature)
        );

        console.log('Final sections before ordering:', allSections.length);

        // PASS 5: Reading Order & Validation
        const validator = new ReadingOrderValidator();
        const { sections, validation } = validator.finalizeAndValidate(
            allSections,
            items,
            columnInfo,
            signature
        );

        console.log('Coverage:', validation.coveragePercent.toFixed(1) + '%');
        console.log('Final sections:', sections.length);

        return {
            sections: sections,
            validation: validation,
            signature: signature,
            metadata: {
                totalTextItems: items.length,
                blockCount: blocks.length,
                sectionCount: sections.length,
                coveragePercent: validation.coveragePercent,
                isValid: validation.isValid
            }
        };
    }
}
```

---

## Key Architectural Improvements

### 1. Separation of Concerns
| Pass | Responsibility | V2 (Broken) | V3 (Fixed) |
|------|----------------|-------------|------------|
| Block Detection | Find visual boundaries | Mixed with break logic | Dedicated pass with percentile thresholds |
| Classification | Determine block type | Per-item regex | Multi-signal scoring on blocks |
| Decomposition | Split into sections | One-size-fits-all rules | Type-specific logic |
| Validation | Check coverage | None | Explicit validation pass |

### 2. Statistical Robustness
**V2:** Fixed multipliers (0.85, 0.60) → Brittle
**V3:** Percentile-based thresholds → Adaptive

**V2:** K-means clustering (sensitive to outliers) → Unreliable
**V3:** Percentile calculation (robust to outliers) → Stable

### 3. Context-Aware Decisions
**V2:** Binary decision per item (no lookahead)
**V3:** Analyze entire block, then decompose

**V2:** "Should I break here?"
**V3:** "What is this block, and how should it be decomposed?"

### 4. Multi-Signal Classification
**V2:** Regex match → Type
**V3:** Score all types → Best match

Example:
```
Block: "1. First item\n2. Second item"

V2: Regex sees "1." → header (WRONG)
V3: Scores header=2, numbered-list=11 → numbered-list (CORRECT)
```

### 5. Explicit Validation
**V2:** No validation, trust output
**V3:** Check coverage, detect overlaps, calculate confidence

---

## Expected Performance

### Test Case 1: Wrong Deal.pdf (Page 2)
**Current V2 Results:** 3-40 sections (varies)
**Expected V3 Results:**
- Pass 2: Detects 6 blocks (bullets, paragraphs, table)
- Pass 3: Classifies bullet block, paragraph blocks, table block
- Pass 4: Splits bullet block into 3 bullet sections
- Pass 5: Final output = 6 sections (P1-P6)
- Coverage: 100%

### Test Case 2: PP Proposal Page 3
**Current V2 Results:** Missing 50-80% of content
**Expected V3 Results:**
- Pass 2: Detects 10+ blocks (each paragraph is a block)
- Pass 3: Classifies header, paragraphs
- Pass 4: Each block → one section
- Pass 5: Final output = 10-12 sections (P1-P12)
- Coverage: 98-100%

### Test Case 3: PP Proposal Page 14
**Current V2 Results:** Over-segmented (each line separate)
**Expected V3 Results:**
- Pass 2: Groups lines into 4 paragraph blocks (using p75 threshold)
- Pass 3: Classifies as body-paragraph blocks
- Pass 4: Each block → one section
- Pass 5: Final output = 4 sections (P1-P4)
- Coverage: 100%

---

## Implementation Priority

### Phase 1: Core Block Detection (Highest Priority)
1. Implement `DocumentAnalyzer` with percentile-based gap analysis
2. Implement `VisualBlockDetector` with adaptive threshold
3. Test on both tight-spaced and loose-spaced documents
4. **Success criteria:** Blocks match visual paragraph boundaries

### Phase 2: Classification
1. Implement `BlockClassifier` with multi-signal scoring
2. Test header vs numbered-list disambiguation
3. Test bullet list detection
4. **Success criteria:** >90% correct type classification

### Phase 3: Decomposition
1. Implement `BlockDecomposer` with type-specific rules
2. Test bullet continuation logic
3. Test numbered list splitting
4. **Success criteria:** Bullets properly grouped, lists properly split

### Phase 4: Validation
1. Implement `ReadingOrderValidator`
2. Add coverage checking
3. Add overlap detection
4. **Success criteria:** 95%+ coverage on all test documents

---

## Fallback Strategies

### If Block Detection Still Struggles
**Fallback:** Use multiple percentile thresholds
- p75 for primary blocks
- p50 for secondary splits within blocks
- p90 for major section boundaries

### If Classification Confidence Low
**Fallback:** Default to body-paragraph type (safe default)
- Still better than V2 because decomposition will handle it correctly

### If Coverage < 95%
**Root cause investigation:**
1. Check uncovered items - are they outliers (headers/footers)?
2. Check if block detection threshold too high
3. Add post-processing step to capture orphan items

---

## Migration Path from V2 to V3

### Step 1: Add V3 alongside V2 (A/B Testing)
- Keep V2 parser intact
- Add V3 parser classes
- Run both, compare outputs
- Visualize both on UI (toggle between parsers)

### Step 2: Validate V3 on Test Cases
- Test on "Wrong Deal.pdf"
- Test on "PP Proposal Comm College.pdf"
- Measure coverage, section count, correctness

### Step 3: Gradual Rollout
- If V3 coverage > V2 coverage → Use V3
- If V3 coverage < V2 coverage → Use V2 (fallback)
- Log metrics for analysis

### Step 4: Full Replacement
- Once V3 consistently outperforms V2 → Remove V2
- Keep V3 as primary parser

---

## Conclusion

**The fundamental problem with V2:** Trying to solve a global optimization problem (section segmentation) with local greedy decisions (gap thresholds).

**The V3 solution:** Build global context first (blocks), then make informed decisions (classification, decomposition).

**Key paradigm shift:**
- V2: "Should I break here?" (local)
- V3: "What are the blocks, and how should each be decomposed?" (global)

This architecture should achieve:
- **95%+ coverage** (every text item in a section)
- **±15% section count** (matches human perception)
- **Robust across diverse documents** (adaptive thresholds)
- **Maintainable** (clear separation of concerns, no magic numbers)

---

**Next Steps:**
1. Review this architecture
2. Identify any concerns or edge cases
3. Begin Phase 1 implementation (core block detection)
4. Test on real documents
5. Iterate based on results
