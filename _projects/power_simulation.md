---
layout: page
title: A Priori Power Simulation
description: Power simulation for the information load experiment
importance: 1
category: work
---
A priori power analysis for "Curiosity And Memory Under Cognitive Load: A Test Of Resource Boundaries And Prediction Error Effects". Simulation-based power analysis was conducted for H1 and H7. Full preregistration including design parameters and model specification available at OSF (currently embargoed, will be publicly accessible upon publication).

Power was estimated using tailored simulation-based sample-size planning following the framework recommended by Pargent et al. (2024) for binomial GLMMs with cross-classified random effects, as formula-based tools (e.g., GPower) are not appropriate for this model structure. A custom simulation function was written in R generating trial-level data matching the study design: In the H1/H2 simulation, load conditions were assigned to fixed question positions rather than randomized per participant as in the actual design. In the H7 simulation, load was randomized per participant across 60 images, matching the actual design. This discrepancy is acknowledged as a minor limitation of the H1/H2 simulation; the confound is unlikely to substantially affect power estimates as random effects variability averages across iterations. Curiosity was operationalised as within-person z-scored ratings from Session 1, carried forward to Session 2. Cross-classified random intercepts were included for participants (σ = 0.50) and questions (σ = 0.50). Load was treated as a categorical factor with load 1 as the reference level, consistent with the planned confirmatory analysis. Two models were fitted per iteration — an additive model and an interaction model — both using glmer with binomial family. Power for H1 (curiosity main effect) was defined as the proportion of iterations in which the curiosity coefficient from the interaction model reached significance at α = .05, capturing the curiosity effect specifically at load 1. Power for H2 (curiosity × load interaction) was defined as the proportion of iterations in which the omnibus likelihood ratio test comparing the additive and interaction models reached significance at α = .05. 200 iterations were run per condition. Beta coefficients were set conservatively below direct meta-analytic conversions (d = 0.44, β ≈ 0.80 for deliberate recall), with β = 0.15 –0.25 adopted to account for paradigm-specific differences and uncertainty in effect size translation. Attenuation of the curiosity effect across load levels was varied across weak, moderate, and strong scenarios, with separate interaction terms specified for each load level. A large simulated dataset (N = 3000) confirmed the assumed parameters produced mean recall accuracy of approximately 40%, 33%, 27%, and 22% at load levels 1, 3, 5, and 7 respectively, consistent with expectations for a challenging trivia recall task under increasing cognitive load. A GPower RM ANOVA approximation using the full meta-analytic effect size (d = 0.44) suggested N = 40–50 as a rough lower bound. However, given the conservative simulation parameters adopted (β = 0.15–0.25), the simulation-based estimates are considered more appropriate for the present design.

Pargent et al. (2024) identify four circumstances under which tailored simulation-based sample-size planning is necessary over existing software packages (see their Table 1). Three of these applied directly to the present design.
First, design complexity: the study design features a varying binomial denominator across load conditions (1, 3, 5, 7 answers per trial), cross-classified random effects for both participants and questions, and within-person z-scoring of curiosity applied prior to model fitting. None of the available packages with full GLMM support — including simr (Green & MacLeod, 2016) and mixedpower (Kumle et al., 2021) — accommodate this combination without customisation that effectively reduces to writing custom code anyway.
Second, absence of pilot data: simr and similar packages typically start from a fitted model object, which requires existing data. As no prior study has used this exact paradigm, all population parameters had to be specified from scratch based on meta-analytic effect sizes and domain knowledge — the approach Pargent et al. explicitly recommend in this situation.
Third, transparency: a custom simulation function makes every assumption about the data-generating process explicit and auditable, which is preferable for a preregistered dissertation where parameter choices must be fully justifiable to reviewers and a dissertation committee.
For these reasons, custom R code using lme4 was used, implementing the same simulation logic as package-based approaches but tailored precisely to the study design. 

RESULTS

## Table 1: H1 & H2 Power Analysis Results (N = 45)

| β curiosity | β attenuation (load 3 / 5 / 7) | Power: H1 | Power: H2 |
|:---:|:---:|:---:|:---:|
| 0.15 | −0.05 / −0.10 / −0.15 | 0.320 | 0.990 |
| 0.20 | −0.03 / −0.06 / −0.09 | 0.530 | 0.785 |
| 0.20 | −0.05 / −0.10 / −0.15 | 0.570 | 1.000 |
| 0.20 | −0.08 / −0.16 / −0.24 | 0.515 | 1.000 |
| 0.25 | −0.05 / −0.10 / −0.15 | 0.730 | 0.995 |

*Note.* Convergence failure rate = 0% across all conditions. Power estimated over 200 simulated datasets per condition.

---

## Table 2: H7 Power Analysis Results (N = 45)

| β curiosity | β attenuation (load 3 / 5 / 7) | Power: H7 | Power: Interaction |
|:---:|:---:|:---:|:---:|
| 0.13 | −0.05 / −0.10 / −0.15 | 0.295 | 0.320 |
| 0.13 | −0.07 / −0.12 / −0.16 | 0.330 | 0.375 |
| 0.15 | −0.05 / −0.10 / −0.16 | 0.370 | 0.465 |
| 0.18 | −0.06 / −0.12 / −0.19 | 0.515 | 0.535 |
| 0.18 | −0.05 / −0.09 / −0.14 | 0.440 | 0.310 |

*Note.* Convergence failure rate = 0% across all conditions. Power estimated over 200 simulated datasets per condition.

Simulation results are presented in Tables 1 and 2 for H1, H2 and H7 respectively. Across all conditions, convergence failure rates were 0%, indicating stable model estimation. For H1/H2, power to detect the curiosity × load interaction ranged from 78.5% under weak attenuation to 100% under moderate and strong attenuation, confirming the design is well powered for the primary confirmatory hypothesis. Power for the curiosity main effect at load 1 was more variable, ranging from 32% at the conservative estimate (β = 0.15) to 73% at the optimistic estimate (β = 0.25), with the moderate estimate (β = 0.20) yielding 51 – 57% depending on assumed attenuation — below the conventional 0.80 threshold. For H7, power to detect the curiosity × load interaction ranged from 32% to 53.5% across all conditions, reflecting the small expected effect size for incidental recognition (β = 0.13 – 0.18) and the sensitivity limitations of the omnibus likelihood ratio test for a 3-parameter categorical interaction with a binary outcome. Power for the curiosity main effect in H7 ranged from 29.5% to 51.5%. These results confirm that the design is well powered to detect the curiosity × load interaction for task-specific recall (H2), but is underpowered for the curiosity main effect across both hypotheses and for the interaction in incidental recognition (H7). Consistent with this, Bayesian GLMMs will be adopted as the primary inferential approach for the curiosity main effect in H1 and H7, and as the sole inferential approach for H7, allowing estimation of load-specific curiosity effects with appropriate uncertainty quantification rather than reliance on binary hypothesis testing decisions the design is insufficiently powered to support. Sample size was fixed at N = 45 across all simulation conditions, reflecting recruitment constraints. Power estimates should therefore be interpreted as the expected power achievable within this sample size rather than as a justification for it. The simulation included a small positive session coefficient (β = 0.05) reflecting a modest practice effect in Session 2. This assumption is acknowledged as a simplification — memory consolidation literature would predict slight decay rather than improvement across sessions. Inspection of the simulation parameters suggested this term has negligible impact on power estimates for the primary hypotheses, and the session term is included in the confirmatory model as a covariate regardless of its direction. Attenuation parameters were not derived from prior literature due to the absence of directly comparable studies, and were instead varied across plausible ranges to assess robustness of power estimates.

```r
library(dplyr)
library(lme4)

get_power_accuracy <- function(
    beta_curiosity = 0.20,
    beta_load3     = -0.30,
    beta_load5     = -0.60,
    beta_load7     = -0.90,
    beta_cur3      = -0.03,
    beta_cur5      = -0.06,
    beta_cur7      = -0.09,
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
        load_f  = factor(load, levels=c(1,3,5,7)),
        load3   = as.integer(load == 3),
        load5   = as.integer(load == 5),
        load7   = as.integer(load == 7),
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
          beta_load3     * load3 +
          beta_load5     * load5 +
          beta_load7     * load7 +
          0.05           * session +
          beta_cur3      * curiosity_wp * load3 +
          beta_cur5      * curiosity_wp * load5 +
          beta_cur7      * curiosity_wp * load7 +
          person_intercept[participant_id] +
          question_intercept[question_id],
        p         = plogis(eta),
        n_correct = rbinom(n(), load, p),
        n_total   = load
      )

    df_trial <- df_ans %>%
      group_by(participant_id, question_id, session,
               curiosity_wp, load_f, load3, load5, load7) %>%
      summarise(n_correct = sum(n_correct),
                n_total   = sum(n_total),
                .groups   = "drop")

    fit_main <- suppressWarnings(tryCatch(
      glmer(cbind(n_correct, n_total - n_correct) ~
              curiosity_wp + load_f + session +
              (1 | participant_id) +
              (1 | question_id),
            family  = binomial,
            data    = df_trial,
            control = glmerControl(optimizer = "bobyqa",
                                   optCtrl   = list(maxfun = 1e5))),
      error = function(e) NULL
    ))

    fit_int <- suppressWarnings(tryCatch(
      glmer(cbind(n_correct, n_total - n_correct) ~
              curiosity_wp * load_f + session +
              (1 | participant_id) +
              (1 | question_id),
            family  = binomial,
            data    = df_trial,
            control = glmerControl(optimizer = "bobyqa",
                                   optCtrl   = list(maxfun = 1e5))),
      error = function(e) NULL
    ))

    if (!is.null(fit_main) && !is.null(fit_int)) {
      coefs  <- tryCatch(summary(fit_int)$coefficients,
                         error = function(e) NULL)
      p_main <- if (!is.null(coefs))
                  coefs["curiosity_wp", "Pr(>|z|)"] else NA

      lrt   <- tryCatch(anova(fit_main, fit_int),
                        error = function(e) NULL)
      p_int <- if (!is.null(lrt)) lrt$"Pr(>Chisq)"[2] else NA

      if (!is.na(p_main) && p_main < 0.05) sig_main        <- sig_main + 1L
      if (!is.na(p_int)  && p_int  < 0.05) sig_interaction <- sig_interaction + 1L

    } else { fails <- fails + 1L }

    if (i %% 50 == 0) cat(sprintf("  [H1/H2] sim %d/%d\n", i, n_sims))
  }

  data.frame(
    hypothesis        = "H1/H2 Accuracy",
    beta_curiosity    = beta_curiosity,
    beta_cur3         = beta_cur3,
    beta_cur5         = beta_cur5,
    beta_cur7         = beta_cur7,
    N                 = n_part,
    power_curiosity   = round(sig_main        / n_sims, 3),
    power_interaction = round(sig_interaction / n_sims, 3),
    conv_fail_pct     = round(fails / n_sims * 100, 1)
  )
}

get_power_h7 <- function(
    beta_curiosity = 0.13,
    beta_load3     = -0.20,
    beta_load5     = -0.40,
    beta_load7     = -0.60,
    beta_cur3      = -0.05,
    beta_cur5      = -0.10,
    beta_cur7      = -0.15,
    n_part         = 45,
    n_sims         = 200) {

  sig_main        <- 0L
  sig_interaction <- 0L
  fails           <- 0L

  for (i in seq_len(n_sims)) {

    person_intercept <- rnorm(n_part, 0, 0.50)
    image_intercept  <- rnorm(60,    0, 0.40)

    df_s1 <- expand.grid(
      participant_id = 1:n_part,
      trial_id      = 1:60
    ) %>%
      group_by(participant_id) %>%
      mutate(
        load    = sample(rep(c(1,3,5,7), each=15)),
        load_f  = factor(load, levels=c(1,3,5,7)),
        load3   = as.integer(load == 3),
        load5   = as.integer(load == 5),
        load7   = as.integer(load == 7),
        session = 0,
        cur_raw = pmin(pmax(rnorm(n(), 3.5, 1.5), 1), 6),
        curiosity_wp = ifelse(sd(cur_raw) > 0,
          (cur_raw - mean(cur_raw)) / sd(cur_raw), 0)
      ) %>%
      ungroup()

    df_s2  <- mutate(df_s1, session = 1)
    df_all <- bind_rows(df_s1, df_s2) %>%
      mutate(
        eta = 0.40 +
          beta_curiosity * curiosity_wp +
          beta_load3     * load3 +
          beta_load5     * load5 +
          beta_load7     * load7 +
          0.05           * session +
          beta_cur3      * curiosity_wp * load3 +
          beta_cur5      * curiosity_wp * load5 +
          beta_cur7      * curiosity_wp * load7 +
          person_intercept[participant_id] +
          image_intercept[trial_id],
        p      = plogis(eta),
        recall = rbinom(n(), 1, p)
      )

    fit_main <- suppressWarnings(tryCatch(
      glmer(recall ~
              curiosity_wp + load_f + session +
              (1 | participant_id) +
              (1 | trial_id),
            family  = binomial,
            data    = df_all,
            control = glmerControl(optimizer = "bobyqa",
                                   optCtrl   = list(maxfun = 1e5))),
      error = function(e) NULL
    ))

    fit_int <- suppressWarnings(tryCatch(
      glmer(recall ~
              curiosity_wp * load_f + session +
              (1 | participant_id) +
              (1 | trial_id),
            family  = binomial,
            data    = df_all,
            control = glmerControl(optimizer = "bobyqa",
                                   optCtrl   = list(maxfun = 1e5))),
      error = function(e) NULL
    ))

    if (!is.null(fit_main) && !is.null(fit_int)) {
      coefs  <- tryCatch(summary(fit_int)$coefficients,
                         error = function(e) NULL)
      p_main <- if (!is.null(coefs))
                  coefs["curiosity_wp", "Pr(>|z|)"] else NA

      lrt   <- tryCatch(anova(fit_main, fit_int),
                        error = function(e) NULL)
      p_int <- if (!is.null(lrt)) lrt$"Pr(>Chisq)"[2] else NA

      if (!is.na(p_main) && p_main < 0.05) sig_main        <- sig_main + 1L
      if (!is.na(p_int)  && p_int  < 0.05) sig_interaction <- sig_interaction + 1L

    } else { fails <- fails + 1L }

    if (i %% 50 == 0) cat(sprintf("  [H7] sim %d/%d\n", i, n_sims))
  }

  data.frame(
    hypothesis        = "H7 Recognition",
    beta_curiosity    = beta_curiosity,
    beta_cur3         = beta_cur3,
    beta_cur5         = beta_cur5,
    beta_cur7         = beta_cur7,
    N                 = n_part,
    power_curiosity   = round(sig_main        / n_sims, 3),
    power_interaction = round(sig_interaction / n_sims, 3),
    conv_fail_pct     = round(fails / n_sims * 100, 1)
  )
}

conditions_h1 <- list(
  list(beta_curiosity=0.20, beta_cur3=-0.03, beta_cur5=-0.06, beta_cur7=-0.09),
  list(beta_curiosity=0.20, beta_cur3=-0.05, beta_cur5=-0.10, beta_cur7=-0.15),
  list(beta_curiosity=0.20, beta_cur3=-0.08, beta_cur5=-0.16, beta_cur7=-0.24),
  list(beta_curiosity=0.15, beta_cur3=-0.05, beta_cur5=-0.10, beta_cur7=-0.15),
  list(beta_curiosity=0.25, beta_cur3=-0.05, beta_cur5=-0.10, beta_cur7=-0.15)
)

conditions_h7 <- list(
  list(beta_curiosity=0.13, beta_cur3=-0.05, beta_cur5=-0.10, beta_cur7=-0.15),
  list(beta_curiosity=0.13, beta_cur3=-0.07, beta_cur5=-0.12, beta_cur7=-0.16),
  list(beta_curiosity=0.15, beta_cur3=-0.05, beta_cur5=-0.10, beta_cur7=-0.16),
  list(beta_curiosity=0.18, beta_cur3=-0.06, beta_cur5=-0.12, beta_cur7=-0.19),
  list(beta_curiosity=0.18, beta_cur3=-0.05, beta_cur5=-0.09, beta_cur7=-0.14)
)

cat("=== H1/H2: ACCURACY SIMULATION ===\n\n")

results_h1 <- do.call(rbind, lapply(conditions_h1, function(x) {
  cat(sprintf("beta_cur=%.2f | attenuation=%.2f/%.2f/%.2f\n",
              x$beta_curiosity, x$beta_cur3, x$beta_cur5, x$beta_cur7))
  do.call(get_power_accuracy, x)
}))

cat("\n=== FINAL RESULTS: H1/H2 ===\n")
print(results_h1, row.names = FALSE)

cat("\n=== H7: RECOGNITION SIMULATION ===\n\n")

results_h7 <- do.call(rbind, lapply(conditions_h7, function(x) {
  cat(sprintf("beta_cur=%.2f | attenuation=%.2f/%.2f/%.2f\n",
              x$beta_curiosity, x$beta_cur3, x$beta_cur5, x$beta_cur7))
  do.call(get_power_h7, x)
}))

cat("\n=== FINAL RESULTS: H7 ===\n")
print(results_h7, row.names = FALSE)

```
Simulation code was initially generated with the assistance of Claude (Anthropic, 2026) and subsequently reviewed, corrected, and validated by the author. All modelling decisions, parameter specifications, and interpretations are the author's own, and the author takes full responsibility for the code as presented.
