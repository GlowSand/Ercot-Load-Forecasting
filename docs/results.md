# Results

![final_comparison](final_comparison.png)

## Main Holdout Year: 2025

| Model                   |     MAE |    RMSE |   MAPE | R_squared |
|------------------------|--------:|--------:|-------:|----------:|
| LightGBM               | 2717.484 | 3816.321 | 4.772% | 0.86073 |
| Baseline_CurrentERCOT  | 3150.300 | 4254.870 | 5.639% | 0.82688 |
| Random Forest          | 3249.169 | 4491.537 | 5.685% | 0.80700 |
| LSTM                   | 3404.534 | 4566.864 | 5.990% | 0.80100 |

## Interpretation of 2025 Results

The 2025 holdout year is the main benchmark in this project because it provides a full unseen annual cycle.

The main result is that LightGBM clearly performed best on this split.

A few things stand out:

- LightGBM produced the lowest MAE, RMSE, and MAPE
- The baseline remained strong, which shows that persistence is a hard benchmark to beat
- Random Forest and LSTM were both competitive, but neither beat LightGBM
- The LSTM did not provide a major advantage over the stronger tabular models for this setup

My takeaway from 2025 is that a well-engineered boosting model generalized best for this task.

---

## Secondary Forward Check: 2026 YTD

| Model                   |     MAE |    RMSE |   MAPE | R_squared |
|------------------------|--------:|--------:|-------:|----------:|
| Baseline_CurrentERCOT  | 2911.989 | 3997.250 | 5.444% | 0.62906 |
| LightGBM               | 3053.558 | 4638.775 | 5.483% | 0.50044 |
| Random Forest          | 3981.828 | 5484.103 | 7.177% | 0.30178 |
| LSTM                   | 3999.069 | 5563.192 | 7.129% | 0.28410 |

## Interpretation of 2026 YTD Results

The 2026 YTD window is not the main benchmark because it is incomplete and does not contain the full year.

Still, it is useful as a forward robustness check.

Here, the most surprising result is that the naive baseline outperformed all learned models.

I interpret this as evidence that:

- very recent persistence remained especially strong in this incomplete window
- the models that generalized best on the full 2025 year did not generalize as strongly to this shorter and more recent regime
- evaluation window choice matters a lot in forecasting

This is one of the most useful findings in the project, because it shows that a model can win on a strong full-year benchmark and still lose to a simple baseline in a more local forward setting.

---

## Model-by-Model Takeaways

### Baseline
The baseline was stronger than I expected. It remained competitive on 2025 and actually won on 2026 YTD.

### Random Forest
Random Forest was useful as a tree-based comparison model, but it did not beat LightGBM on the full 2025 holdout and degraded sharply on 2026 YTD.

### LightGBM
LightGBM was the strongest overall model on the main holdout year and is the model I would treat as the best primary forecasting model in this project.

### LSTM
The LSTM was competitive but not superior. It did not beat LightGBM on the main holdout year, which reinforces the idea that deep learning is not automatically the best choice for structured load-weather forecasting or tabular data.

---

## Overall Conclusion

My main conclusion is:

- LightGBM is the best model on the full 2025 holdout year
- Baseline persistence is the strongest model on the incomplete 2026 YTD forward check

That combination is important because it shows that forecasting quality depends heavily on the evaluation window, and that strong baselines are essential for honest model comparison.