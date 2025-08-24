# Backtrader Essentials - Book Outline

## Chapter 1: Backtrader Basics & Data Feeds

- [Setting the Stage: Imports](chapter_1_subsections/01_setting_the_stage_imports.md)
- [Acquiring Historical Market Data](chapter_1_subsections/02_acquiring_historical_market_data.md)
- [Feeding Data into Backtrader](chapter_1_subsections/03_feeding_data_into_backtrader.md)
- [The Cerebro Engine: Setting the Stage](chapter_1_subsections/04_cerebro_engine_setting_stage.md)
- [Adding a Minimal Strategy and Running the Test](chapter_1_subsections/05_minimal_strategy_running_test.md)
- [Visualizing Results: Plotting](chapter_1_subsections/06_visualizing_results_plotting.md)
- [Putting It All Together: Full Script](chapter_1_subsections/07_putting_it_all_together.md)
- [Chapter Summary](chapter_1_subsections/08_chapter_summary.md)

## Chapter 2: Built-In Indicators

- [Indicators as Strategy Building Blocks](chapter_2_subsections/01_indicators_as_strategy_building_blocks.md)
- [The Simple Moving Average (SMA) - Concept](chapter_2_subsections/02_simple_moving_average_concept.md)
- [Implementing SMA in Backtrader](chapter_2_subsections/03_implementing_sma_in_backtrader.md)
- [Accessing SMA Values](chapter_2_subsections/04_accessing_sma_values.md)
- [SMA Use Case: Crossover Logic Examples](chapter_2_subsections/05_sma_crossover_logic_examples.md)
- [The Relative Strength Index (RSI) - Concept](chapter_2_subsections/06_rsi_concept.md)
- [Implementing RSI in Backtrader](chapter_2_subsections/07_implementing_rsi_in_backtrader.md)
- [RSI Use Case: Overbought/Oversold Signal Logic](chapter_2_subsections/08_rsi_overbought_oversold_signals.md)
- [Chaining Indicators: Smoothing the RSI](chapter_2_subsections/09_chaining_indicators_smoothing_rsi.md)
- [Hands-On: The MyStrategyWithIndicators Code](chapter_2_subsections/10_hands_on_strategy_with_indicators.md)
- [Code Walkthrough & Cerebro Integration](chapter_2_subsections/11_code_walkthrough_cerebro_integration.md)
- [Chapter Summary](chapter_2_subsections/12_chapter_summary.md)

## Chapter 3: Multi-Line Indicators

- [Introduction: Beyond Single Lines](chapter_3_subsections/01_introduction_beyond_single_lines.md)
- [Moving Average Convergence Divergence (MACD) - Concept](chapter_3_subsections/02_macd_concept.md)
- [MACD - Purpose and Signals](chapter_3_subsections/03_macd_purpose_and_signals.md)
- [Implementing MACD in Backtrader](chapter_3_subsections/04_implementing_macd_in_backtrader.md)
- [Bollinger Bands (BBands) - Concept & Purpose](chapter_3_subsections/05_bollinger_bands_concept_purpose.md)
- Implementing Bollinger Bands in Backtrader
- [Hands-On: MyMultiLineIndicatorStrategy Walkthrough](chapter_3_subsections/07_hands_on_multiline_indicator_strategy.md)
- [Chapter Summary](chapter_3_subsections/08_chapter_summary.md)

## Chapter 4: Trend Strength – ADX System

- [Introduction: Measuring Trend Strength](chapter_4_subsections/01_introduction_measuring_trend_strength.md)
- [Understanding the Directional Indicators: +DI and -DI](chapter_4_subsections/02_understanding_directional_indicators_di.md)
- [Interpretation of +DI and -DI](chapter_4_subsections/03_interpretation_of_di_indicators.md)
- [Understanding the ADX Line: The Trend Strength Gauge](chapter_4_subsections/04_understanding_adx_trend_strength_gauge.md)
- [Implementing the ADX System in Backtrader](chapter_4_subsections/05_implementing_adx_system_in_backtrader.md)
- [Full Example Code: AdxStrategyExample](chapter_4_subsections/06_full_example_code_adx_strategy.md)
- [Code Walkthrough: AdxStrategyExample](chapter_4_subsections/07_code_walkthrough_adx_strategy.md)
- [Chapter Summary](chapter_4_subsections/08_chapter_summary.md)

## Chapter 5: Mean-Reversion & Momentum

- [Introduction: Mean Reversion vs. Momentum](chapter_5_subsections/01_introduction_mean_reversion_vs_momentum.md)
- [Bollinger Bands Strategy: Mean Reversion Approach](chapter_5_subsections/02_bollinger_bands_mean_reversion_approach.md)
- [RSI Strategy: Overbought/Oversold Reversals](chapter_5_subsections/03_rsi_overbought_oversold_reversals.md)
- [MACD Strategy: Momentum/Trend Crossover Approach](chapter_5_subsections/04_macd_momentum_trend_crossover_approach.md)
- [MACD Strategy Discussion & Comparative Context](chapter_5_subsections/05_macd_strategy_discussion_comparative_context.md)
- [Chapter Summary](chapter_5_subsections/06_chapter_summary.md)

## Chapter 6: Custom Indicators

- [Introduction: Extending Backtrader's Arsenal](chapter_6_subsections/01_introduction_extending_backtrader_arsenal.md)
- [Anatomy of a Custom Indicator: Structure](chapter_6_subsections/02_anatomy_of_custom_indicator_structure.md)
- [Minimum Period & Rate of Change (ROC) Example - Concept & Code](chapter_6_subsections/03_minimum_period_roc_example_concept_code.md)
- [Using the Custom Indicator in a Strategy](chapter_6_subsections/04_using_custom_indicator_in_strategy.md)
- [Chapter Summary](chapter_6_subsections/05_chapter_summary.md)

## Chapter 7: Combining Signals

- [Introduction: The Quest for Robust Signals](chapter_7_subsections/01_introduction_quest_for_robust_signals.md)
- [Filtering Example: SMA Crossover + RSI](chapter_7_subsections/02_filtering_example_sma_crossover_rsi.md)
- [SmaCrossFiltered - Walkthrough](chapter_7_subsections/03_smacrossfiltered_walkthrough.md)
- [Beyond Signal Reversal: Position Management & Exit Rules](chapter_7_subsections/04_beyond_signal_reversal_position_management.md)
- [Applying Exit Concepts & Best Practices Introduction](chapter_7_subsections/05_applying_exit_concepts_best_practices.md)
- [Advanced Filtering and Exits: SMA + RSI + ADX + ATR + Bracket Orders](chapter_7_subsections/06_advanced_filtering_exits_multi_indicator.md)
- [Implementation (Enhanced SmaCrossFiltered)](chapter_7_subsections/07_implementation_enhanced_strategy.md)
- [Advanced Strategy Walkthrough](chapter_7_subsections/08_advanced_strategy_walkthrough.md)
- [Best Practices for Signal Robustness](chapter_7_subsections/09_best_practices_signal_robustness.md)
- [Chapter Summary](chapter_7_subsections/10_chapter_summary.md)

## Chapter 8: Analyzers, Optimization & Next Steps

- [Introduction: Evaluating and Improving](chapter_8_subsections/01_introduction_evaluating_improving.md)
- [Measuring Performance with Analyzers](chapter_8_subsections/02_measuring_performance_analyzers.md)
- [Key Built-in Analyzers](chapter_8_subsections/03_key_builtin_analyzers.md)
- [Interpreting Analyzer Results](chapter_8_subsections/04_interpreting_analyzer_results.md)
- [Introduction to Parameter Optimization](chapter_8_subsections/05_introduction_parameter_optimization.md)
