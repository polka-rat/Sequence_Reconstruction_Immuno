# Experimental Methodology

## Trial Procedure and Error Estimation

### 3.1 Experimental Design

Our analysis evaluates the performance of sequence reconstruction algorithms across different communication channels by determining the minimum number of traces (N) required to recover a transmitted sequence with high reliability. We conducted systematic trials for the Bit-Wise Mode (BWM) algorithm and the Prefix Filtering Mode (PFM) across multiple channels: W1, W2, and W3.

### 3.2 Trial Structure

Each trial follows a standardized protocol:

1. **Sequence Generation**: A random binary sequence X of length n is generated, where each bit is independently drawn from a Bernoulli(1/2) distribution.

2. **Channel Simulation**: The sequence X is transmitted through a selected channel model N times, generating N corrupted traces. Each trace represents a realization of the channel's stochastic behavior.

3. **Reconstruction Algorithm**: The estimation algorithm (either BWM or PFM) processes the N traces to produce a reconstructed sequence estimate $\hat{X}$.

4. **Error Detection**: We record whether the reconstructed sequence exactly matches the original sequence. A trial results in an error if $\hat{X} \neq X$ and a success if $\hat{X} = X$.

### 3.3 Adaptive Sampling Strategy

Rather than using a fixed number of trials, we employ an adaptive sampling approach to efficiently explore the parameter space while collecting sufficient error statistics:

**Phase 1: Coarse Scanning**
- Begin with a moderate number of traces
- Continue until the error probability drops below the target threshold (e.g., 0.01 for delta experiments)

**Phase 2: Threshold Refinement**
- Once a threshold region is identified, backtrack to the approximate crossover point
- Reduce step size to refine precision 
- Reset and re-scan with higher resolution to precisely locate the performance boundary

This two-phase approach balances computational efficiency with precision, avoiding wasteful trials in regions where performance is clearly well above or below the target.

### 3.4 Error Probability Estimation

For each (channel, n, N) or (channel, n, delta) combination, we conduct multiple independent trials:

$$\hat{p}_{\text{error}} = \frac{\text{Number of errors}}{{\text{Total number of trials}}}$$

**Trial Count Strategy**: We maintain adaptive trial counts with a minimum error threshold:
- Minimum errors collected: 30 errors per (channel, N) pair
- Maximum trials: 100,000 to prevent excessive computation
- Batch size: 100 trials per computational batch to enable parallelization

This ensures stable error estimates while accommodating the varying error rates across different N values—requiring fewer trials when error rates are high and more trials when approaching the threshold.

### 3.5 Confidence Interval Calculation

We compute 95% confidence intervals (CI) for error probability estimates using the **Wilson Score Method** from the binomial test:

$$\text{CI}_{\text{Wilson}} = \left[\frac{p' - z_{\alpha/2}\sqrt{p'(1-p')/n}}{1 + z_{\alpha/2}^2/n}, \frac{p' + z_{\alpha/2}\sqrt{p'(1-p')/n}}{1 + z_{\alpha/2}^2/n}\right]$$

where $p'$ is the observed error rate, $n$ is the sample size, and $z_{\alpha/2} = 1.96$ for 95% confidence.

The Wilson method is preferred over normal approximation because it:
- Provides accurate intervals even for extreme error rates
- Maintains nominal coverage probability across all parameter ranges
- Avoids intervals outside [0, 1] for boundary cases

### 3.6 Parallel Processing

Trials within each batch are executed in parallel using joblib to accelerate computation:

```python
Parallel(n_jobs=-1)(
    delayed(run_single_trial)(X, N, n, channel_func, estimate_func) 
    for X in X_trials
)
```

Setting `n_jobs=-1` utilizes all available processor cores, significantly reducing wall-clock time while maintaining statistical independence of trials.

### 3.7 Performance Metrics

We extract three performance thresholds from our trial data:

1. **N_Central**: The minimum N where the point estimate of error probability drops below the target
2. **N_Lower_CI**: The conservative bound where the upper confidence limit drops below the target
3. **N_Upper_CI**: The optimistic bound where the lower confidence limit drops below the target

These three estimates provide complementary views of performance:
- **N_Central**: The "best estimate" of required traces
- **N_Lower_CI**: A statistically conservative estimate suitable for guarantees
- **N_Upper_CI**: An optimistic estimate for favorable conditions

### 3.8 Experimental Parameters

**For N vs n analysis** (Figure showing N vs sequence length):
- n values: 12 to 50 (step of 2)
- Target error probability: < 0.01
- Algorithms: Bitwise Majority Voting
- Channels: W1, W2, W3

**For N vs Delta analysis** (Figure showing N vs error probability target):
- Sequence length (n): Fixed at 20
- Delta targets: 50 logarithmically-spaced values from 0.01 to 0.5
- Algorithms: Bitwise Majority Voting, Progressive Filtering Method
- Channels: W1, W2

### 3.9 Statistical Validation

All reported results include 95% confidence intervals computed from binomial trials. The adaptive sampling ensures:
- **Minimum statistical power**: At least 30 errors per estimate
- **Computational efficiency**: Early stopping when bounds converge
- **Robustness**: Confidence intervals account for sampling variability
- **Reproducibility**: All randomness seeded and trial counts recorded

This methodology provides reliable, statistically grounded estimates of algorithm performance across a comprehensive range of experimental conditions.
