{component symbol} is `DW01A`.
{documentation folder} is `kicad-happy-review/components/DW01A`


For a given electronic component: **{component symbol}**, with documentation available in folder: `{documentation folder}`, prepare a comprehensive application summary.

The folder structure for each component contains the main datasheet along with subfolders holding additional supporting documents — read and incorporate those as well.
The Markdown documents may contain references to schematics, diagrams, and other images — analyze only those that appear within sections relevant to this task (e.g., electrical specifications, application circuits, layout guidelines). Ignore images found in unrelated sections such as packaging, mechanical drawings, or soldering instructions.

The summary should include at least the following sections (where available in the source documents):

- **Short description** of the component's purpose and role;
- **Electrical requirements**: maximum, minimum, and typical operating conditions;
- **Sample applications** and reference circuits;
- **Component placement, routing, and layout best practices**;

**Do not include:**
- Packaging information;
- Soldering tips or guidelines;
- Physical dimensions;
- Component programming or software usage — unless it directly affects the electrical application. If a specific use or programming mode requires a particular electrical configuration, mention that configuration only.

Format the analysis as a **tidy, structured Markdown document** using bullet points and tables where appropriate. Use the component symbol as the filename and save it in the folder: `kicad-happy-review/docs_summaries`.

This document will later be used as a basis to validate an electronic board design in which this component is used.

