# DSC232
Link to dataset: https://github.com/smousavi05/STEAD

---

### 3. Fitting Analysis (4 points)

- Where does your model fit in the fitting graph (underfitting vs. overfitting)?
    - Our RF Regressor model is underfitting, while our XGBoost Regressor was severely overfitting.
- Build at least one model with **different hyperparameters** and compare results
    - We have one Random Forest Regressor model with 20 `num_trees` and `max_depth` 5. Another RF Regressor model with 40 `num_trees` and `max_depth` 3. For XGBoost Regressor we had the hyperparameter set as `max_depth` = 8 with `eta` = 0.1 .
    - as a result, our baseline model with 20 trees with max_depth 5 had a better performance 
        - RF Regressor max_depth = 5; 20 trees: test RMSE: 87.47; train RMSE: 84.71
        - RF Regressor max_depth = 3; 40 trees: test RMSE: 93.55
        - XGBoost Regressor max_depth = 8; eta: 0.1; test RMSE: 431.35559602907716
- Which model performs best and why?
    - as of now, our Random Forest Regressor with max_depth = 5 and 20 trees is showing the best performance as it is showing the lowest test RMSE score while the training and testing has a similar score as well.
- What are the next models you are thinking of for Milestone 4 and why?
    - As our data is a time series heavy dataset, we might be looking into some time series applicable models such as GRU cells or a model named PhaseNet.
    - When we were still predicting `trace_category` rather than `s_arrival_sample`, we were running into a lot of data leakage issues with the model learning from intentional null values, and we were strongly considering Linear ODE for that scenario. We may look into if it still makes sense to use Linear ODE for our current setup.

---

### 4. Conclusion Section (5 points)

Write a conclusion for your first model:

- What is the conclusion of your 1st model?
- What can be done to possibly improve it?
- How did distributed computing help with this task?

### 5. Speedup Analysis (5 points)

**Waveform Frequency Downsampling**
| Executors | Time (sec) | Speedup | Efficiency |
|---|---|---|---|
| 1 | 3.11 | 1.00x | 100% |
| 7 | 2.83 | 1.10x | 16% |

---

**Waveform Array to Vector**
| Executors | Time (sec) | Speedup | Efficiency |
|---|---|---|---|
| 1 | 3.48 | 1.00x | 100% |
| 7 | 3.46 | 1.01x | 14.4% |

---

**Pipeline (VectorAssembler)**
| Executors | Time (sec) | Speedup | Efficiency |
|---|---|---|---|
| 1 | 447.48 | 1.00x | 100% |
| 7 | 450.98 | 0.99x | 14.2% |

---

**Evaluation Runtime**
| Executors | Time (sec) | Speedup | Efficiency |
|---|---|---|---|
| 1 | 362.94 | 1.00x | 100% |
| 7 | 370.58 | 0.98x | 14% |
