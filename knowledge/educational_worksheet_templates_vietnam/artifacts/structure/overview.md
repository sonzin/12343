# HTML-based Educational Worksheets Structure

This document outlines the structural patterns used for creating printable educational worksheets, specifically tailored for elementary education (Grade 2 in Vietnam).

## General Page Setup

The worksheets are designed to be printed, typically on A4 paper.

### CSS Styling for Print and Display

Modern versions use CSS variables for theme consistency and specialized containers for A4 page simulation. The `.page` class is designed to handle multi-page worksheets.

```css
:root {
    --primary-color: #1a1a1a;
    --accent-color: #2c3e50;
    --bg-body: #f8f9fa;
    --bg-page: #ffffff;
    --border-color: #333;
}

* {
    box-sizing: border-box;
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
}

body {
    font-family: 'Quicksand', sans-serif;
    background-color: var(--bg-body);
    margin: 0;
    padding: 20px;
}

.page {
    width: 210mm;
    min-height: 297mm;
    padding: 20mm;
    margin: 20px auto;
    background: var(--bg-page);
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
    position: relative;
    page-break-after: always; /* Essential for multi-page printing */
}

@media print {
    body { background: none; padding: 0; }
    .page { margin: 0; box-shadow: none; width: 100%; height: 100%; padding: 15mm; }
    .no-print { display: none; }
}
```

## Header Layout

The header contains the worksheet title, subject/topic, and student information fields.

```html
<div class="header">
    <h1>📚 PHIẾU BÀI TẬP TOÁN LỚP 2</h1>
    <p style="font-size: 12pt;">Chủ đề: Phép nhân</p>
    <div class="info">
        <span><b>Họ tên:</b> <span class="dot"></span></span>
        <span><b>Lớp:</b> <span class="dot" style="min-width:50px"></span></span>
        <span><b>Ngày:</b> <span class="dot" style="min-width:80px"></span></span>
    </div>
</div>
```

### Student Info Dots
The `.dot` class creates an underlined space for handwritten input:
```css
.dot {
    border-bottom: 1px solid #000;
    min-width: 130px;
}
```

## Sectioning

Worksheets are typically divided into two main parts:
1. **TRẮC NGHIỆM (Multiple Choice)**: Concept checking.
2. **TỰ LUẬN (Written/Problem Solving)**: Practice and application.

```html
<div class="section">I. TRẮC NGHIỆM</div>
<!-- Questions here -->
<div class="section">II. TỰ LUẬN</div>
<!-- Exercises here -->
```

### Section CSS
```css
.section {
    background: #000;
    color: #fff;
    padding: 6px 15px;
    font-weight: bold;
    margin: 15px 0 10px;
    font-size: 13pt;
}
```
