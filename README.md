# portfolio-optimization-deep-reinforcement-learning

This project investigates portfolio allocation strategies in the Chinese pharmaceutical and biotechnology industry by comparing classical optimization approaches with modern reinforcement learning methods.

The study evaluates:

- Equal Weight (EW)
- Mean–Variance Optimization (Sharpe Ratio Maximization)
- Risk Parity (Equal Risk Contribution)
- Deep Reinforcement Learning (Proximal Policy Optimization, PPO)

A custom OpenAI Gym environment was developed to simulate dynamic portfolio allocation with transaction costs and rebalancing penalties. The reinforcement learning agent learns adaptive asset weights under non-stationary market conditions.

## Key Features

- Quantitative portfolio optimization framework
- Risk contribution analysis and performance comparison
- Custom RL trading environment
- PPO implementation using Stable-Baselines3
- Backtesting on China biotech sector equities (2018–2025)

## Motivation

Traditional portfolio optimization methods rely on static assumptions and are sensitive to estimation errors. This project explores whether reinforcement learning can improve adaptability and robustness in volatile sector-specific markets.

## Author

Weixun Xie (Stephanie Xie)  
M.A. Statistics — Columbia University
