clear; clc;
close all;

%% 1. Parameter Definitions
S0 = 100;      % Initial asset price (Spot price)
K = 105;       % Strike price
T = 1;         % Time to maturity (Years)
r = 0.05;      % Risk-free interest rate
sigma = 0.25;  % Volatility
steps = 252;   % Time steps (Number of trading days)
dt = T/steps;  % Time step size (dt)

% Test scenarios (Number of simulations for error analysis)
N_trials = [1e3, 1e4, 1e5, 1e6]; 

%% 2. Algorithm and Simulation (Vectorized Approach) 
fprintf('--- Simulation Started ---\n');

% Run simulation for the highest N (For maximum precision)
N = N_trials(end);
tic; % Start performance timer 

% Random Walk (Geometric Brownian Motion)
% Calculate all simulations simultaneously for each time step
Z = randn(N, steps); 
S = zeros(N, steps + 1);
S(:,1) = S0;

for t = 1:steps
    S(:,t+1) = S(:,t) .* exp((r - 0.5*sigma^2)*dt + sigma*sqrt(dt)*Z(:,t));
end

toc_time = toc;
fprintf('Execution Time: %.4f seconds\n', toc_time);

%% 3. Option Pricing Models
%European Call Option
ST = S(:, end);
payoff_call = max(ST - K, 0);
price_call = exp(-r*T) * mean(payoff_call);

%% 4. Analytical Validation (Black-Scholes) 
d1 = (log(S0/K) + (r + 0.5*sigma^2)*T) / (sigma*sqrt(T));
d2 = d1 - sigma*sqrt(T);
N_d1 = 0.5 * (1 + erf(d1 / sqrt(2)));
N_d2 = 0.5 * (1 + erf(d2 / sqrt(2)));
BS_Price = S0 * N_d1 - K * exp(-r*T) * N_d2;

%% 5. Visualization and Reporting
figure('Color', 'w');

% Chart 1: Price Paths (First 100 scenarios)
subplot(2,1,1);
plot(S(1:100, :)', 'LineWidth', 0.5);
hold on;
plot(mean(S), 'k', 'LineWidth', 3); % Mean path
title('Simulated Asset Price Paths (GBM)');
xlabel('Time Steps'); ylabel('Price');

% Chart 2: Distribution at Maturity
subplot(2,1,2);
histogram(ST, 100, 'FaceColor', [0.7 0.7 0.9]);
xline(K, 'r', 'LineWidth', 2, 'Label', 'Strike Price');
title('Price Distribution at Maturity');
xlabel('Price'); ylabel('Frequency');

fprintf('\n--- Results ---\n');
fprintf('European Call (MC): %.4f\n', price_call);
fprintf('Black-Scholes:     %.4f\n', BS_Price);
