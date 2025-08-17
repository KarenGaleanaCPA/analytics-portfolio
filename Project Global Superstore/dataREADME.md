# Data Notes & Attribution

## Source
- **Dataset:** Global Superstore (public sample retail dataset)
- **Origin:** Kaggle — [Global Superstore](https://www.kaggle.com/datasets/shekpaul/global-superstore)
- **Intended use:** Educational/portfolio demonstration for analytics workflow.

> This repository currently **includes the full Kaggle dataset** for convenience and educational/portfolio use. Please review the dataset’s license/terms on Kaggle before redistribution or forking.

## Local Layout
```
data/
  raw/        # place the original GlobalSuperstore.xls (or .xlsx) here (excluded from git)
  sample/     # tiny, sanitized subset committed for reproducibility
  processed/  # CSVs exported by the notebook (fact + dimension tables)
```

## Transformations (summary)
- Header standardization and type casting
- Guardrails (e.g., clamp discounts to [0,1], safe divide for margin)
- Derived fields (ship lead time, profit margin, dates)
- Optional winsorization for viz-only copies
- Final star schema exports for BI (fact + dims)

