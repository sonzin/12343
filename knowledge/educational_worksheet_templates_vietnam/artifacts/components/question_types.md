# Worksheet Components

This library documents common UI components used in educational worksheets.

## 1. Multiple Choice Question (`.q`)

A container for a question and its options.

```html
<div class="q">
    <span class="q-num">Câu 1:</span> Question text here?
    <div class="opts">
        <span class="opt">A. Option 1</span>
        <span class="opt">B. Option 2</span>
        <span class="opt">C. Option 3</span>
        <span class="opt">D. Option 4</span>
    </div>
</div>
```

### Styling
- `.q`: Bordered box with padding.
- `.opts`: Flex container for options.
- `.opt`: Rounded capsule for each choice.

## 2. Visual Math Groups (`.img-box`)

Used for multiplication or counting exercises.

```html
<div class="img-box">
    <div class="row">
        <div class="grp"><span class="obj lg">🍎🍎🍎</span><small>(3 quả)</small></div>
        <div class="grp"><span class="obj lg">🍎🍎🍎</span><small>(3 quả)</small></div>
    </div>
</div>
```

## 3. Matching Game (`.match`)

Two columns of items intended to be connected by lines.

```html
<div class="match">
    <div class="match-col">
        <div class="match-item">4 × 3</div>
        <div class="match-item">5 × 4</div>
    </div>
    <div style="display:flex;align-items:center;font-size:28pt;font-weight:bold">⟷</div>
    <div class="match-col">
        <div class="match-item">12</div>
        <div class="match-item">20</div>
    </div>
</div>
```

## 4. Problem Solving Block (`.prob`)

Displays a word problem with lines for the solution.

```html
<div class="prob">
    <div class="prob-title">a. Word problem text here...</div>
    <div class="sol-line"></div>
    <div class="sol-line"></div>
    <b>Đáp số:</b> <span class="dot" style="min-width:120px"></span>
</div>
```

## 5. Math Formula with Blanks (`.formula`)

Used for fill-in-the-blank math equations.

```html
<div class="pic-math">
    <!-- Visual context -->
    <div class="formula">
        <span class="blank"></span> × <span class="blank"></span> = <span class="blank"></span>
    </div>
</div>
```

### Blank Input Styling
```css
.blank {
    width: 40px;
    height: 32px;
    border: 2px solid #000;
    display: inline-block;
}
```
## 6. Calculation Chain (`.chain-container`)

Used for multi-step logic paths with visual icons (arrows, circles, hexagons).

```html
<div class="chain-container">
    <div class="circle">2</div>
    <div class="arrow">
        <span class="op-text">× 5</span>
        <div class="arrow-line"></div>
    </div>
    <div class="hexagon"></div>
</div>
```

## 7. Solution Grid (`.drawing-grid`)

Simulates math paper grids for students to write solutions or solve problems.

```html
<div class="drawing-grid">
    <div class="grid-cell"></div><div class="grid-cell"></div>...
</div>
```

## 8. Icon-based Question Headers (`.q-icon`)

Numbered circles often used for exercise headings.

```html
<div class="q-title">
    <div class="q-icon">1</div> 
    Đếm thêm 2 rồi viết số thích hợp vào chỗ chấm.
</div>
```

## 9. Problem Image Integration (`.problem-img`)

Used to embed custom illustrations within word problems or exercise blocks.

```html
<div class="problem-box">
    <div class="problem-img">
       <img src="img/example_image.png" alt="Description">
    </div>
    <p>Word problem text goes here...</p>
</div>
```

## 10. Side-by-Side Content Layout

A flex-based layout to place text instructions on one side and an illustration on the other.

```html
<div style="display: flex; gap: 20px;">
    <div style="flex: 1;">
        <p>Text/Instruction content here...</p>
        <!-- inputs/math steps -->
    </div>
    <div style="width: 250px; text-align: center;">
        <img src="img/side_illustration.png" style="width: 100%;">
    </div>
</div>
```
## 11. Composite Counting Problem

Multi-step problems involving different multipliers (e.g., animals with different number of legs).

```html
<div class="problem-box">
    <div style="display: flex; gap: 15px;">
        <div style="flex: 1;">
            <p>Trong sân có 6 con gà và 2 con chó. Hỏi có tất cả bao nhiêu cái chân?</p>
            <div class="sol-line"></div>
            <div class="sol-line"></div>
        </div>
        <div style="width: 200px;">
            <img src="img/animals.png" style="width: 100%;">
        </div>
    </div>
</div>
```
