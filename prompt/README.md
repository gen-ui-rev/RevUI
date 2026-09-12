# Generation prompts

This directory provides the prompt templates for source UI generation, revision instruction generation, and revised alternatives generation. Related rules, conditional additions, and user-message templates are grouped by generation step.

The templates are edited for the public release and are not verbatim API request logs. They use the terminology and request fields of the released dataset. Per-example inputs are represented by named placeholders.

## Files and generation flow

```text
prompt/
├── README.md
├── source-ui-generation/
│   └── generation.txt
├── instruction/
│   ├── 01-applicability.txt
│   ├── 02-change-specification.txt
│   ├── 03-request-generation.txt
│   └── 04-instruction-assembly.txt
└── alternatives/
    └── generation.txt
```

| Step | Template | Input → output |
|---|---|---|
| Source UI | [source-ui-generation/generation.txt](source-ui-generation/generation.txt) | Screen category, application domain, visual attributes, and canvas → one HTML interface. |
| Applicable change targets | [instruction/01-applicability.txt](instruction/01-applicability.txt) | Source HTML and seven change-target definitions → eligible targets. |
| Change specification | [instruction/02-change-specification.txt](instruction/02-change-specification.txt) | Source HTML and one to three assigned targets → jointly designed five-field specifications. |
| Individual requests | [instruction/03-request-generation.txt](instruction/03-request-generation.txt) | One specification and its assigned specificity level → one request. |
| Instruction assembly | [instruction/04-instruction-assembly.txt](instruction/04-instruction-assembly.txt) | Ordered requests → one combined revision instruction. |
| Revised alternatives | [alternatives/generation.txt](alternatives/generation.txt) | Source HTML, revision instruction, and canvas → alternatives A and B in one response. |

## How to read and use the templates

Each TXT file contains a `SYSTEM` block and a `USER TEMPLATE` block, followed by the options and conditional rules used to assemble them. Section headings and assembly notes organize the files; they are not additional model messages. Insert only the options whose conditions apply.

Replace named placeholders such as `{source_html}`, `{category}`, or `{instruction}` with the corresponding values. JSON examples and CSS declarations are literal content, not substitution fields. When an optional block does not apply, replace its placeholder with an empty string. When inserting a line such as `{now_line}`, include its trailing newline so the following field begins on a new line.

### Source UI generation

The source UI template combines a screen category, its definition and boundary, an application domain, content density, accent color, separation treatment, corner radius, imagery strategy, typeface, and canvas. The category taxonomy and visual assignments are per-example inputs; the template does not enumerate all assignment values.

Choose the full-screen or component canvas block for `{canvas_line}`. Full-screen UIs use 390 × 844 CSS pixels. Component UIs use a width of 390 pixels and content-driven heights between 240 and 560 pixels. Select the matching scope and imagery text. Insert the photo rules only for photographic imagery; otherwise leave `{photo_block}` empty. Typeface instructions receive the assigned font stack and, when applicable, the exact font links. The optional repair message carries validation findings, with the removal/simplification note added only for overflow issues.

### Revision instruction generation

1. Determine applicable change targets using the seven definitions in `01-applicability.txt`.
2. Supply the assigned targets to `02-change-specification.txt`. Each specification contains `element`, `now`, `goal`, `solution`, and `axis`. For multiple requests, include the set-level rules so the changes are designed together and can be evaluated separately. Include the device rule when Environment (F) is assigned. This step does not receive the specificity level.
3. Use `03-request-generation.txt` once per specification. Insert the shared level rules, the applicable target rules, and the assigned word limits. Include the extra E rule at L4. The output is an object with an `en` field.
4. If there are two or three requests, assemble their `en` values in the order returned by the specification step using `04-instruction-assembly.txt`. A single request is used directly without an assembly call. The combined output also uses `en`.

| Level | Information to express | Length target |
|---|---|---|
| L1 | Element, position, and form of the solution. | 10–18 words |
| L2 | What to change and what to do, leaving position and form open. | 5–11 words |
| L3 | Desired outcome, leaving the element and means open. | 5–11 words |
| L4 | What changes and the kind of change, without fixing the outcome. | 5–9 words |

These levels constrain the **request to be written**. The request-generation template still receives `element`, `goal`, `solution`, and `axis` at every level; `now` is included only at L1–L3. The level rules control which information may appear in the generated request. Environment changes retain the target device at every level, including L4. For Situation, Context & Scenario (E) at L4, the new audience or subject is left unspecified.

### Revised alternatives generation

Supply the source HTML, the final revision instruction as `{instruction}`, and the applicable canvas description. For a device change, provide the target device and its assigned dimensions. Otherwise use the original canvas dimensions.

One model generates **both A and B in a single response**. The prompt asks for meaningfully different approaches wherever the request leaves room for interpretation. It does not assign different models or predetermined strengths to A and B. Each alternative is returned as a complete HTML document under `=== VARIANT A ===` or `=== VARIANT B ===`.
