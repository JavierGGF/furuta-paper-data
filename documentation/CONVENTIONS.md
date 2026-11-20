# Signal and model conventions

## Telemetry
`samples.csv` contains integer `sequence`, device timestamp `device_us` (microseconds), referenced `arm_counts` and `pend_counts`, instantaneous signed software `applied_pwm`, status bit field `flags`, and fault/error counters. Other fields are retained unchanged for traceability. `received_at` is a host monotonic reception timestamp, not a calendar date or actuation instant. `phase` is acquisition bookkeeping. `local_state.csv` contains device timestamps, `mode` (2 means CAPTURE), controller diagnostics and state-message fields. A record alias is an acquisition session, not a model.

Outputs in fitting/prediction coordinates are [theta, alpha] in radians: theta=2*pi*arm_counts/921.6; alpha=mod(2*pi*pend_counts/4096,2*pi)-pi. Theta is inferred from the motor encoder; alpha=0 is upright. u=applied_pwm/32. This is not measured torque or voltage. Nominal sample period is0.01s; retain sample order without resampling.

## Assessment selection
Associate each DATA row with the latest preceding state row. Require CAPTURE mode2, state age0..30000 microseconds, (flags AND130)=130, faults/pend_errors/motor_errors all0, abs(alpha)<=5deg and abs(theta)<=20deg. Split at invalid rows, nonconsecutive sequence counters (modulo2**32), or intersample intervals outside8000..12000 microseconds. Retain pieces of at least45 samples. Segments are numbered from0 in acquisition order. The exact resulting windows are supplied.

Within each segment of length L, origins are k=8+m*h with m>=0 and k<L-h. Targets are k+h. Report nominal horizons h=1,10,20,50 as10,100,200,500ms. Figure2 uses the broader recorded CAPTURE trajectory, not only assessment-eligible windows; the final10s precede the last armed sample.

## Linear models
For lag order na, the regressor is [theta[k],alpha[k],theta[k-1],alpha[k-1],...,theta[k-na+1],alpha[k-na+1],u[k-1]]. Each JSON coefficient matrix has2*na+1 rows and2 output columns, already in original radian/input coordinates. Compute next output=regressor times coefficient; do not reapply RMS scaling. There is no intercept. Historical OLS8=Original-OLS8; OLS4-A=Matched-OLS4-A; OLS8-A=Matched-OLS8-A; Ridge4-A=Frozen-Ridge4-A.

Initialize from measured history at origin. At recursive step ell=1..h, R uses u[k+ell-2]; H uses u[k-1]; C uses u[k-1] for ell1 and u[k] afterwards. No later output measurement updates a forecast. Persistence holds the measured output at origin. Endpoints are compared to the same recorded trajectory; held commands were not physically applied.

## Exploratory models
Ridge4-12 is the shared R1+R2 base (neural LINEAR_MODELS.json key Matched-Ridge4, residual RIDGE_MODEL.json, options BASELINE_RIDGE.json). DirectPast18 has a separate9x2 map for each h. Use the same four-lag regressor and no future commands. Coefficients are in original coordinates. The multistep model retains the same recurrence.

Neural parameter vector of length26 is flattened in row-major order: first18 values are a9x2 hidden-weight matrix H; next2 hidden biases beta; next4 a2x2 output-weight matrix V; final2 output biases zeta. Standardize x with saved xmean/xscale; g=tanh(z@H+beta)@V+zeta. Standalone output=g*yscale+ymean. Residual output=x@coefficient+gain*g*yscale, without adding an output mean. Recursively update predicted histories. Median neural results are medians of five separate seed-specific RMSEs, not ensemble forecasts.

## Metrics
Errors in the per-window CSV are prediction minus measured target, in degrees. Record RMSE=sqrt(mean(error**2)) across all eligible endpoint windows, separately for each coordinate. For the ECDF sort absolute errors and assign cumulative percentages100*i/N for i=1..N. Do not interpret these percentages as confidence intervals. CV-A fits a slope to eight measured samples with their actual timestamps and extrapolates from the latest measured output to actual target time; CV-N uses nominal h*0.01s.
