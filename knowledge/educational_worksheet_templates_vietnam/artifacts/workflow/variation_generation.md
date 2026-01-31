# Variation Generation Workflow

This document describes the workflow for generating new worksheet versions based on existing templates and image references.

## The "Preserve Structure, Update Content" Pattern

When a user provides images of a worksheet (like a workbook page) but asks for a "similar" task, the workflow should prioritize structural fidelity while randomizing data to prevent identical duplication.

### 1. Structural Fidelity
- Maintain all UI components documented in `components/question_types.md`.
- Keep the CSS layout (A4 `.page` container, `.q-icon` headers, tables).
- Preserve the *type* of task (e.g., if Question 1 is a counting pattern, Question 1 in the new version must also be a counting pattern).

### 2. Content Variation Logic
- **Numerical Scales**: Shift the starting point or the step (e.g., from 2, 4, 6... to 10, 12, 14...).
- **Factor Permutation**: In multiplication tables, shuffle the order of factors or use different pairs within the same target family (Bảng nhân 2/5).
- **Word Problem Subject Swap**:
    - Chị Lan -> Bạn An
    - Len/Mũ -> Giỏ/Táo
    - Bút chì -> Cái kẹo
- **Chain Logic**: Update the starting number and operations to result in a different final sum, while keeping the same number of steps and visual flow.

### 3. Prompting for Success
When requesting variations, use instructions such as:
> "Dựa trên cấu trúc phiếu bài tập hiện tại, hãy soạn một đề mới với các phép tính khác hoàn toàn để bé không bị lặp lại, nhưng vẫn giữ nguyên các dạng câu hỏi và trình bày chuyên nghiệp."

## Example: Before vs. After Variation

| Item | Original (Image) | Variation (New Version) |
| :--- | :--- | :--- |
| **Counting** | 2; 4; 6... 20 | 10; 12; 14... 28 |
| **Word Problem** | 4 rolls of yarn, 5 hats each | 8 baskets of apples, 5 apples each |
| **Logic Chain** | 2 -> x5 -> -5 -> x8 = 40 | 5 -> x2 -> +5 -> x2 = 30 |

## 4. AI Illustration Generation

To enhance visual engagement, especially for word problems, use generative AI tools to create custom assets:

### Illustration Prompting Patterns
- **Standard**: "A cute educational illustration for a 2nd grade math worksheet. [Subject Description]. Clean line art with soft colors, white background, vector style."
- **Example (Baskets)**: "A cute educational illustration for a 2nd grade math worksheet. 8 simple woven baskets arranged in rows, each basket contains exactly 5 red apples. Clean line art with soft colors, white background, vector style."
- **Example (Candies)**: "A cute educational illustration for a 2nd grade math worksheet. A collection of 20 colorful wrapped candies. Clean line art with soft colors, white background, vector style."

### Integration Workflow
1.  **Generate**: Use the prompt patterns above.
2.  **Organize**: Save images in a local `img/` directory relative to the HTML file.
3.  **Implement**: Embed using `<img>` tags within `.problem-img` or side-by-side flex containers.
4.  **Refine**: If the user suggests a logic correction (e.g., clearing a value in a calculation chain), update the HTML but keep the surrounding visual assets.
## 5. Difficulty Scaling (Advanced Thinking)

To challenge students and enhance critical thinking, upgrade basic multiplication/division tasks into multi-step or composite problems.

### Scaling Strategies
- **Composite Counting**: Instead of counting one type of object, mix multiple types with different multipliers (e.g., "6 chickens and 2 puppies. How many legs in total?"). This combines multiplication (6x2 and 2x4) with addition.
- **Reverse Operations**: Provide the result and some parts of the formula, asking the student to find the missing operation or factor.
- **Hidden Logic**: Use sequences that involve two different operations in alternating steps (e.g., +2, then +5).
- **Partitioning**: Ask students to divide a total into multiple groups in more than one way.

### Prompting AI for "Advanced" Content
When asking for a harder version, specify the cognitive load:
> "Tạo một phiên bản nâng cao với các bài toán tổ hợp (ví dụ: tính tổng số chân của nhiều loại động vật khác nhau) để bé phải suy luận qua 2 bước tính."

### Illustration Adjustments for Advanced Problems
Ensure illustrations clearly show the features needed for the advanced task (e.g., "Clear distinction of legs" in the animal prompt).
