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

- RF Regressor num_trees: 20; max_depth: 5 test statistics graph
<img width="1590" height="590" alt="image" src="https://github.com/user-attachments/assets/12ced923-a516-4b79-a219-102353e7d5c2" />

- RF Regressor num_trees: 40; max_depth: 3 test statistics graph
<img width="1590" height="590" alt="image" src="https://github.com/user-attachments/assets/003329ba-648c-4477-8455-3e827f1f0085" />

-  The left graph of both models reveals two clusters. The graph of RF Regressor num_trees: 20; max_depth: 5 has more spread out clusters, indicating that it's predictions were more varied than that of num_trees: 40; max_depth: 3. The cluster that hovers above predicted s-wave sample value of 200 is very narrow for the latter model, but that model has a slightly higher test RMSE, suggesting that it did not quite narrow in on the correct value.

    - **RMSE Interpretation**: The original data is collected in 100hz. However, we down sampled the data to 20hz instead. Due to do this, our evaluation for RF Regressor max_depth = 5; 20 trees will be having a offset of 87.47 * 0.05 = around 4.37 seconds from the actual `s_arrival time`.
- Which model performs best and why?
    - as of now, our Random Forest Regressor with max_depth = 5 and 20 trees is showing the best performance as it is showing the lowest test RMSE score while the training and testing has a similar score as well.
- What are the next models you are thinking of for Milestone 4 and why?
    - As our data is a time series heavy dataset, we might be looking into some time series applicable models such as GRU cells or a model named PhaseNet.
    - When we were still predicting `trace_category` rather than `s_arrival_sample`, we were running into a lot of data leakage issues with the model learning from intentional null values, and we were strongly considering Linear ODE for that scenario. We may look into if it still makes sense to use Linear ODE for our current setup.

---

### 4. Conclusion Section (5 points)

Our Random Forest Regressor model demonstrated the most reliable performance among all evaluated models, achieving test and train RMSEs of very close values, avoiding overfitting. However, the test RMSE of 87.47 samples, which is equivalent to an average time offset of 4.37 seconds (87.47 x 0.05s) from the true S-wave arrival time is indicative of underfitting. In seismology, especially considering the downsampling that we did, a 4.37 second window is not small, so our current model is not complex enough to fully capture the high-frequency characteristics embedded in the waveforms.

To improve it, we could have reduced how much we downsampled by. i.e. by 50Hz instead of 20Hz or keeping it at the original 100Hz. We only did this to try and reduce dimensionality. Additionally, we could have tried to increase the maxDepth and numTrees hyperparameters to capture more nuanced relationships within the data. Lastly, there are numerous types of feature engineering methods specific to seismic data that we found through research, such as STA/LTA (Short-Term Average / Long-Term Average).

The speedup analysis indicates that we weren't able to optimize well enough, thus not making full use of distributed computing to help is in this task. There is the possibility that the bottleneck we are clearly experiencing has something to do with the the way MLLib is performing this.

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

---

Compared to the theoretical maximum speedup of x7, our cumulative speedup was 0.99x.

- Waveform frequency downsampling
    - 1.1x speedup with ~10.5% estimated parallelizable fraction of code
- array to vector
    - 1.01x speedup with ~0.7% estimated parallelizable fraction of code
- pipeline (VectorAssembler)
    - 0.99x speedup with ~-0.9% estimated parallelizable fraction of code
- Evaluation runtime
    - 0.98x speedup with ~-2.5% estimated parallelizable fraction of code
