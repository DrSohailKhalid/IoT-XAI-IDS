# Contributing

Please open an issue describing a reproducible problem or a proposed change.
For a bug, include the command, configuration, dataset release and file schema,
Python/package versions, and the complete error message. Do not attach restricted
datasets, credentials, or private traffic records.

For a contribution, create a branch, make a focused change, and open a pull
request explaining the behavior and validation. Run the lightweight checks:

```bash
python -m pip install -r requirements-test.txt
python -m pytest -q
```

The lightweight checks do not train TensorFlow models or run SHAP/LIME. Changes
to those stages need a separate execution check with the training dependencies.
Document any changes to class mapping, splits, feature handling, or optimization
objectives because these can change the scientific interpretation of results.

Retain existing authorship and third-party attribution. Add paper metadata only
when it is confirmed by the authors and supported by the publication record.
