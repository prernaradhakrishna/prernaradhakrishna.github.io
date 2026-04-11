---
layout: page
title: A Priori Power Simulation
description: Power simulation for the information load experiment
importance: 1
category: research
---
A priori power analysis for "Curiosity And Memory Under Cognitive Load: A Test Of Resource Boundaries". Simulation-based power analysis was conducted for H1 and H7. Full preregistration including design parameters and model specification available at OSF (currently embargoed, will be publicly accessible upon publication).

Power was estimated using tailored simulation-based sample-size planning following the framework recommended by Pargent et al. (2024) for binomial GLMMs with cross-classified random effects, as formula-based tools (e.g., G * Power) are not appropriate for this model structure. The target effect was the within-person curiosity coefficient predicting binomial recall accuracy across load conditions. A custom simulation function was written in R generating trial-level data matching the study design: 60 questions across four load conditions (1, 3, 5, 7 answers), within-person z-scored curiosity ratings from Session 1, and cross-classified random intercepts for participants (σ = 0.50) and questions (σ = 0.50). A large simulated dataset (N = 3000) confirmed the assumed parameters produced mean recall accuracy of approximately 38–41% across load conditions, consistent with expectations for a trivia recall task. Each simulated dataset was fitted using `glmer` with binomial family, matching the planned confirmatory analysis. Power was defined as the proportion of iterations in which the curiosity coefficient reached significance at α = 0.05. 200 iterations were run per sample size condition. Simulations were run across conservative, moderate, and optimistic β values derived from a meta-analytic effect size of d = 0.44 (attenuated to β = 0.15 – 0.25), reflecting uncertainty in the true effect size. For H2, which predicts that curiosity-related memory benefits will be attenuated under high cognitive load, such that the recall advantage for high- versus low-curiosity trials is reduced as load increases, the interaction coefficient (curiosity × load) was additionally varied across the same range but with a negative sign, consistent with the resource dependency hypothesis. This allowed power to be estimated not only for the curiosity main effect but also for the detection of a meaningful attenuation pattern across load conditions, under different assumptions about how strongly load suppresses the curiosity benefit.
This was complemented by a G * Power RM ANOVA approximation (Maas & Hox, 2005) as a cross-check, with convergent results across methods supporting a target N of 45 – 50.

Pargent et al. (2024) identify four circumstances under which tailored simulation-based sample-size planning is necessary over existing software packages (see their Table 1). Three of these applied directly to the present design.
First, design complexity: the study design features a varying binomial denominator across load conditions (1, 3, 5, 7 answers per trial), cross-classified random effects for both participants and questions, and within-person z-scoring of curiosity applied prior to model fitting. None of the available packages with full GLMM support — including simr (Green & MacLeod, 2016) and mixedpower (Kumle et al., 2021) — accommodate this combination without customisation that effectively reduces to writing custom code anyway.
Second, absence of pilot data: simr and similar packages typically start from a fitted model object, which requires existing data. As no prior study has used this exact paradigm, all population parameters had to be specified from scratch based on meta-analytic effect sizes and domain knowledge — the approach Pargent et al. explicitly recommend in this situation.
Third, transparency: a custom simulation function makes every assumption about the data-generating process explicit and auditable, which is preferable for a preregistered dissertation where parameter choices must be fully justifiable to reviewers and a dissertation committee.
For these reasons, custom R code using lme4 was used, implementing the same simulation logic as package-based approaches but tailored precisely to the study design. 

RESULTS

## Table 1: H1 & H2 Power Analysis Results (N = 45)

| β (curiosity) | β (curiosity × load) | Power: Main Effect | Power: Interaction |
|:---:|:---:|:---:|:---:|
| 0.15 | −0.05 | 0.525 | 1.000 |
| 0.20 | −0.03 | 0.740 | 1.000 |
| 0.20 | −0.05 | 0.700 | 1.000 |
| 0.20 | −0.08 | 0.745 | 1.000 |
| 0.25 | −0.05 | 0.885 | 1.000 |

*Note.* Convergence failure rate = 0% across all conditions. Power estimated over 200 simulated datasets per condition.

---

## Table 2: H7 Power Analysis Results (N = 45)

| β (curiosity) | β (curiosity × load) | Power: Curiosity | Power: Load | Power: Interaction |
|:---:|:---:|:---:|:---:|:---:|
| 0.13 | −0.03 | 0.405 | 0.945 | 0.600 |
| 0.13 | −0.05 | 0.350 | 0.975 | 0.940 |
| 0.13 | −0.08 | 0.385 | 0.985 | 1.000 |
| 0.17 | −0.05 | 0.550 | 0.960 | 0.965 |
| 0.20 | −0.05 | 0.670 | 0.970 | 0.935 |

*Note.* Convergence failure rate = 0% across all conditions. Power estimated over 200 simulated datasets per condition.

Simulation results are presented in Tables 1 and 2 for H1 and H7 respectively. Across all conditions, convergence failure rates were 0%, indicating stable model estimation. For H1, power to detect the curiosity × load interaction was uniformly high (power = 1.00) regardless of the assumed interaction coefficient, reflecting the strong sensitivity of the design to load-related moderation effects. Power for the curiosity main effect was more variable, ranging from 0.525 at the conservative estimate (β = 0.15) to 0.885 at the optimistic estimate (β = 0.25), with the moderate estimate yielding power of 0.70–0.745 depending on the assumed interaction coefficient — marginally below the conventional 0.80 threshold. For H7, a similar pattern emerged: power for the load main effect and the curiosity × load interaction was consistently high (0.94–1.00) across all conditions, whereas power for the curiosity main effect remained lower, ranging from 0.35 to 0.67 even at the most optimistic coefficient values. These results confirm that the design is well powered to detect load effects and the curiosity × load interaction, but is likely underpowered to detect the curiosity main effect at the smaller effect sizes reported in the literature. Consistent with this, Bayesian GLMMs are adopted as the primary inferential approach for the curiosity effect in H7, where underpowering is most pronounced, allowing estimation of effect magnitude and uncertainty rather than reliance on binary hypothesis testing decisions. Sample size was fixed at N = 45 across all simulation conditions, reflecting recruitment constraints. Power estimates should therefore be interpreted as the expected power achievable within this sample size rather than as a justification for it.

```r
library(dplyr)
library(lme4)

# =============================================================================
# H1/H2: Curiosity predicts accuracy (multi-item, binomial outcome)
# =============================================================================

get_power_accuracy <- function(beta_curiosity = 0.13,
                                beta_cur_load  = -0.05,
                                n_part         = 45,
                                n_sims         = 200) {

  sig_main        <- 0L
  sig_interaction <- 0L
  fails           <- 0L

  for (i in seq_len(n_sims)) {

    person_intercept   <- rnorm(n_part, 0, 0.50)
    question_intercept <- rnorm(60,    0, 0.50)

    df_s1 <- expand.grid(
      participant_id = 1:n_part,
      question_id   = 1:60
    ) %>%
      mutate(
        load    = rep(c(1,3,5,7), each=15)[question_id],
        load_c  = load - 4,
        session = 0,
        cur_raw = pmin(pmax(rnorm(n(), 3.5, 1.5), 1), 6)
      ) %>%
      group_by(participant_id) %>%
      mutate(
        curiosity_wp = ifelse(sd(cur_raw) > 0,
          (cur_raw - mean(cur_raw)) / sd(cur_raw), 0)
      ) %>%
      ungroup()

    df_s2  <- mutate(df_s1, session = 1)
    df_all <- bind_rows(df_s1, df_s2)

    df_ans <- df_all[rep(seq_len(nrow(df_all)), df_all$load), ] %>%
      mutate(
        eta = -0.50 +
          beta_curiosity * curiosity_wp +
          -0.12          * load_c +
          0.05           * session +
          beta_cur_load  * curiosity_wp * load_c +
          person_intercept[participant_id] +
          question_intercept[question_id],
        p         = plogis(eta),
        n_correct = rbinom(n(), load, p),
        n_total   = load
      )

    df_trial <- df_ans %>%
      group_by(participant_id, question_id, session, curiosity_wp, load_c) %>%
      summarise(n_correct = sum(n_correct),
                n_total   = sum(n_total),
                .groups   = "drop")

    fit <- suppressWarnings(tryCatch(
      glmer(cbind(n_correct, n_total - n_correct) ~
              curiosity_wp * load_c + session +
              (1 | participant_id) +
              (1 | question_id),
            family  = binomial,
            data    = df_trial,
            control = glmerControl(optimizer = "bobyqa",
                                   optCtrl   = list(maxfun = 1e5))),
      error = function(e) NULL
    ))

    if (!is.null(fit)) {
      coefs <- tryCatch(summary(fit)$coefficients, error = function(e) NULL)
      if (!is.null(coefs)) {
        p_main <- coefs["curiosity_wp",        "Pr(>|z|)"]
        p_int  <- coefs["curiosity_wp:load_c", "Pr(>|z|)"]
        if (!is.na(p_main) && p_main < 0.05) sig_main        <- sig_main + 1L
        if (!is.na(p_int)  && p_int  < 0.05) sig_interaction <- sig_interaction + 1L
      }
    } else { fails <- fails + 1L }

    if (i %% 50 == 0) cat(sprintf("  [Accuracy] sim %d/%d\n", i, n_sims))
  }

  data.frame(
    hypothesis        = "H1/H2 Accuracy",
    beta_curiosity    = beta_curiosity,
    beta_cur_load     = beta_cur_load,
    N                 = n_part,
    power_curiosity   = round(sig_main        / n_sims, 3),
    power_load        = NA,
    power_interaction = round(sig_interaction / n_sims, 3),
    conv_fail_pct     = round(fails / n_sims * 100, 1)
  )
}

# =============================================================================
# H7: Curiosity predicts incidental information recall (binary recognition)
# =============================================================================

get_power_recall <- function(beta_curiosity = 0.13,
                              beta_cur_load  = -0.05,
                              n_part         = 45,
                              n_sims         = 200) {

  sig_main        <- 0L
  sig_load        <- 0L
  sig_interaction <- 0L
  fails           <- 0L

  for (i in seq_len(n_sims)) {

    person_intercept <- rnorm(n_part, 0, 0.50)
    image_intercept  <- rnorm(60,    0, 0.40)

    df_s1 <- expand.grid(
      participant_id = 1:n_part,
      trial_id      = 1:60
    ) %>%
      mutate(
        load    = rep(c(1,3,5,7), each=15)[trial_id],
        load_c  = load - 4,
        session = 0,
        cur_raw = pmin(pmax(rnorm(n(), 3.5, 1.5), 1), 6)
      ) %>%
      group_by(participant_id) %>%
      mutate(
        curiosity_wp = ifelse(sd(cur_raw) > 0,
          (cur_raw - mean(cur_raw)) / sd(cur_raw), 0)
      ) %>%
      ungroup()

    df_s2  <- mutate(df_s1, session = 1)
    df_all <- bind_rows(df_s1, df_s2) %>%
      mutate(
        eta = 0.40 +
          beta_curiosity * curiosity_wp +
          -0.10          * load_c +
          0.05           * session +
          beta_cur_load  * curiosity_wp * load_c +
          person_intercept[participant_id] +
          image_intercept[trial_id],
        p      = plogis(eta),
        recall = rbinom(n(), 1, p)
      )

    fit <- suppressWarnings(tryCatch(
      glmer(recall ~
              curiosity_wp * load_c + session +
              (1 | participant_id) +
              (1 | trial_id),
            family  = binomial,
            data    = df_all,
            control = glmerControl(optimizer = "bobyqa",
                                   optCtrl   = list(maxfun = 1e5))),
      error = function(e) NULL
    ))

    if (!is.null(fit)) {
      coefs <- tryCatch(summary(fit)$coefficients, error = function(e) NULL)
      if (!is.null(coefs)) {
        p_main <- coefs["curiosity_wp",        "Pr(>|z|)"]
        p_load <- coefs["load_c",              "Pr(>|z|)"]
        p_int  <- coefs["curiosity_wp:load_c", "Pr(>|z|)"]
        if (!is.na(p_main) && p_main < 0.05) sig_main        <- sig_main + 1L
        if (!is.na(p_load) && p_load < 0.05) sig_load        <- sig_load + 1L
        if (!is.na(p_int)  && p_int  < 0.05) sig_interaction <- sig_interaction + 1L
      }
    } else { fails <- fails + 1L }

    if (i %% 50 == 0) cat(sprintf("  [Recall] sim %d/%d\n", i, n_sims))
  }

  data.frame(
    hypothesis        = "H7 Recall",
    beta_curiosity    = beta_curiosity,
    beta_cur_load     = beta_cur_load,
    N                 = n_part,
    power_curiosity   = round(sig_main        / n_sims, 3),
    power_load        = round(sig_load        / n_sims, 3),
    power_interaction = round(sig_interaction / n_sims, 3),
    conv_fail_pct     = round(fails / n_sims * 100, 1)
  )
}

# =============================================================================
# Run both sweeps over the same conditions
# =============================================================================

conditions <- list(
  c(0.13, -0.03),
  c(0.13, -0.05),
  c(0.17, -0.05),
  c(0.20, -0.05),
  c(0.13, -0.08)
)

cat("=== RUNNING H1/H2: ACCURACY SIMULATION ===\n\n")
results_accuracy <- do.call(rbind, lapply(conditions, function(x) {
  cat(sprintf("beta_curiosity = %.2f | beta_cur_load = %.2f\n", x[1], x[2]))
  get_power_accuracy(beta_curiosity = x[1], beta_cur_load = x[2])
}))

cat("\n=== RUNNING H7: INCIDENTAL RECALL SIMULATION ===\n\n")
results_recall <- do.call(rbind, lapply(conditions, function(x) {
  cat(sprintf("beta_curiosity = %.2f | beta_cur_load = %.2f\n", x[1], x[2]))
  get_power_recall(beta_curiosity = x[1], beta_cur_load = x[2])
}))

# =============================================================================
# Print and save
# =============================================================================

results_all <- rbind(results_accuracy, results_recall)

cat("\n=== FINAL RESULTS: H1/H2 ACCURACY ===\n")
print(results_accuracy, row.names = FALSE)

cat("\n=== FINAL RESULTS: H7 INCIDENTAL RECALL ===\n")
print(results_recall, row.names = FALSE)

write.csv(results_all, "power_results.csv", row.names = FALSE)
saveRDS(results_all,   "power_results.rds")
cat("\nResults saved to power_results.csv and power_results.rds\n")
```
Simulation code was initially generated with the assistance of Claude (Anthropic, 2026) and subsequently reviewed, corrected, and validated by the author. All modelling decisions, parameter specifications, and interpretations are the author's own, and the author takes full responsibility for the code as presented.
