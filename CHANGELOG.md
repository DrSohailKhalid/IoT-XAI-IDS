# Changelog

## GitHub preparation, 2026-10-24

Based on the supplied reference implementation identified as version 1.0.0.
This preparation is not a new tagged software release.

- Reworked the README around installation, execution, scope, and navigation.
- Added project-specific upload instructions and a publication checklist.
- Added lightweight GitHub Actions checks and a test-only dependency file.
- Added an executable synthetic smoke check with no benchmark-data download.
- Updated the Colab setup path to match the new repository folder name.
- Expanded ignore rules and added text-file handling rules.
- Added contribution guidance, validation notes, and interpretation limits.
- Clarified CSV expectations, attack-only classification, pooled splits, and
  the absence of an independently verified paper DOI or software DOI.

Authors, author order, supplied affiliations, MIT license, configuration
settings, and software version metadata are retained from the submitted archive.
Any source-code fixes made during validation are recorded separately below.

### Source-code fix

- Replaced the 2D-CNN padding Lambda with built-in Reshape and ZeroPadding1D
  layers. The trailing-zero padding is equivalent; the saved model can be
  loaded using normal Keras defaults. Added an optional TensorFlow regression
  test that checks prediction equality after saving and loading.
