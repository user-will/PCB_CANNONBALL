### Converting PDFs to Markdown
- install [Marker](https://pypi.org/project/marker-pdf/)
- run i.e: `marker_single ~/Downloads/tcan3413.pdf --output_format markdown --output_dir ./kicad-happy-review/components/ --force_ocr`
Having gfx card like Nvidia RTX speedup converting.
Source PDFs and already converted are in files: `pdfs.zip` and `converted-pdfs-to-markdown.zip`

### Making component summary for schematic/pcb review
- edit `kicad-happy-review/ai_prompts/component_summary.md` and prompt LLM with it

### Making kicad-happy review
- install [kicad-happy](https://github.com/aklofas/kicad-happy) skills
- for better tokens usage, consider installing https://github.com/rtk-ai/rtk
- the review process relies on good documentation, so make sure you have it (above steps)
- then in your LLM cli use this prompt: `/kicad Do review using @kicad-happy-review/ai_prompts/circuit_pcb_review.md`
  Example stats of review session - result is `kicad-happy-review/CANNONBALL_review.md`:
  Total duration (API):  21m 31s
  Total duration (wall): 33m 20s
  Total code changes:    215 lines added, 0 lines removed
  Usage by model:
    claude-opus-4-8:  69.5k input, 92.4k output, 4.2m cache read, 315.9k cache write 
