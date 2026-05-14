# 💪 FitFuel — Your Adaptive Fitness Coach

FitFuel is an intelligent, adaptive fitness web application that unifies **nutrition tracking** and **workout planning** into a single platform. Unlike conventional fitness apps that treat diet and exercise as separate concerns, FitFuel recognizes that these are deeply interconnected — and uses **machine learning** to continuously optimize both based on the user's real-world progress.

Built with [Streamlit](https://streamlit.io/), [Supabase](https://supabase.com/), [scikit-learn](https://scikit-learn.org/) and [Claude Opus 4.6](https://claude.ai/login).

> **Live Demo:** [fitfuel.streamlit.app](https://fitfuel-demo.streamlit.app)

---

## The Problem

Most fitness apps fall into one of two categories: workout trackers or nutrition trackers. Users are forced to juggle separate tools that don't communicate with each other, even though nutrition and training are fundamentally linked. A workout plan without aligned nutrition targets leads to stalled progress, and a diet plan that ignores training load leads to under- or over-fueling.

Additionally, most plans are static. They don't adapt when a user consistently struggles with a prescribed volume, over-eats their calorie target, or changes their schedule. This rigidity is one of the primary reasons people abandon their fitness routines.

**FitFuel solves this** by combining both domains into one adaptive system that learns from the user's logged data and explicit feedback to continuously refine its recommendations.

---

## Features

### User Onboarding & Profile Management
New users complete a comprehensive survey covering body metrics, activity level, training experience, goals, equipment access, and physical limitations. This data feeds directly into the plan generation pipeline. Returning users can update their profile at any time, which triggers automatic recalculation of all targets.

### Nutrition Plan Generation
The app calculates each user's Basal Metabolic Rate (BMR) using the revised Harris-Benedict equation, derives Total Daily Energy Expenditure (TDEE) via an activity multiplier, and applies a goal-based calorie adjustment (15% deficit for fat loss, 15% surplus for muscle gain, or maintenance). Macronutrient targets (protein, carbs, fat) are then split according to goal-specific ratios grounded in exercise science literature.

### Workout Plan Generation
A rule-based engine generates personalized weekly training splits (Full Body, Upper/Lower, or Push/Pull/Legs) based on the user's preferred training frequency, experience level, available equipment, and physical limitations. Each exercise is filtered through five criteria — muscle group, equipment availability, limitation conflicts, goal suitability, and difficulty level — before being prescribed with experience-adjusted sets and reps.

### Daily Tracking
- **Nutrition Tracking:** Users log meals with a description, calorie count, macronutrient breakdown, and an optional photo. A real-time dashboard shows progress toward daily targets via interactive gauge charts.
- **Workout Tracking:** Users view their daily prescribed workout and log actual reps completed per set for each exercise. Completion percentage is calculated and visualized to provide immediate feedback.

### Adaptive Machine Learning
After accumulating sufficient data (≥ 7 days), FitFuel's ML pipeline analyzes the user's workout completion rates, calorie adherence, macronutrient consistency, and weight trajectory. A Decision Tree Regressor (scikit-learn) is trained on the user's own historical data to identify trends and predict optimal adjustments. The system modifies calorie targets, macronutrient splits, and training volume — then logs every change with a human-readable explanation for full transparency.

When insufficient data is available (cold start), the system falls back to rule-based heuristics derived from exercise science principles.

### Feedback-Driven Adjustment
Users can submit weekly feedback surveys indicating workout difficulty, nutrition satisfaction, preferred training days, focus areas, and any new physical limitations. This feedback is combined with the ML analysis to produce holistic plan adjustments that respect both data-driven insights and subjective user experience.

---

## AI Transparency

The idea behind FitFuel is entirely our own, we wanted to build something that genuinely doesn't exist on the market: a single app that connects macro-calibrated nutrition tracking with adaptive workout planning in a meaningful, data-driven way. That concept, the problem identification, the feature set, and the overall product vision came from us, not from AI.

That said, we are economics students, and nobody in our team had major exposure to coding before. We used Claude 4.6 as a coding partner throughout development to help translate our vision into working Python code. We want to be transparent about that because we think it reflects how modern software is increasingly built — AI as a tool in the hands of someone who knows what they want to create but is still building their technical skillset.

What's important to understand is what AI did and didn't do in this project. Claude helped us write syntactically correct code, structure files, and implement specific functions. But it couldn't make the decisions that actually shaped the app. We validated that the Harris-Benedict equation and TDEE multipliers used in the nutrition engine are grounded in established exercise science. We researched and selected the macro ratios per goal based on sports nutrition literature. We attempted to move all API credentials into a `.env` file with `.gitignore` exclusion to follow secure development practices — the implementation caused runtime issues on Streamlit Cloud, so we reverted, but the effort reflects our understanding that credentials should not live in source code in a production environment.

Most critically, we handled the integration work ourselves. AI can generate individual functions, but connecting Supabase to the Streamlit frontend, debugging authentication flows, ensuring data written during onboarding is correctly read by the dashboard and workout pages, resolving schema mismatches between what the code expects and what the database returns — that end-to-end wiring is something we worked through manually, and it deepened our understanding of how a full-stack application actually fits together.

Sections of the codebase where AI contributed to the implementation are marked with inline comments indicating AI involvement. The overall architecture, feature priorities, UX decisions, and domain-specific validation remain ours.

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | Streamlit | Interactive web UI with multi-page navigation |
| Database | Supabase (PostgreSQL) | User profiles, workout plans, nutrition/workout logs, feedback, adjustment history |
| Visualization | Plotly | Gauge charts, trend lines, stacked bars, pie charts, progression tracking |
| Machine Learning | scikit-learn | Decision Tree Regressor for adaptive plan optimization |
| Data Processing | Pandas, NumPy | Feature engineering and data aggregation |
| Image Handling | Pillow | Meal photo processing |

---

## Project Structure

```
fitfuel/
├── app.py                        # Main entry point — onboarding, profile, feedback, ML adaptation
├── pages/
│   ├── 1_Dashboard.py            # Daily progress gauges, trend charts, weight tracking
│   ├── 2_Nutrition.py            # Meal logging, macro breakdown, calorie adherence charts
│   └── 3_Workout.py              # Daily workout view, rep tracking, strength progression
├── utils/
│   ├── config.py                 # Exercise database, constants, Supabase credentials
│   ├── calculations.py           # BMR, TDEE, and macronutrient calculations
│   ├── supabase_client.py        # All database read/write operations
│   ├── workout_engine.py         # Rule-based workout plan generation
│   └── ml_model.py               # Adaptive ML model (Decision Tree pipeline)
├── requirements.txt              # Python dependencies
└── .streamlit/
    └── config.toml               # Dark theme configuration
```

---

## Database Schema

FitFuel uses 7 tables in Supabase:

| Table | Purpose |
|-------|---------|
| `user_profiles` | Onboarding data, body metrics, calculated nutrition targets |
| `workout_plans` | Weekly training plans stored as JSON, with active/inactive flagging |
| `workout_logs` | Per-exercise performance logs with actual reps per set |
| `nutrition_logs` | Meal entries with calories, macros, and optional photo URLs |
| `weight_logs` | Weight measurements over time for trend analysis |
| `user_feedback` | Weekly feedback survey responses with processing flags |
| `plan_adjustments` | Audit trail of ML-driven plan changes with before/after values |

---

## Machine Learning Approach

The adaptive system uses a **Decision Tree Regressor** trained on 8 engineered features derived from the user's daily logs:

1. **Workout completion rate** — average across all exercises per day
2. **Calorie adherence ratio** — actual intake vs. target
3. **Protein adherence ratio**
4. **Carb adherence ratio**
5. **Fat adherence ratio**
6. **Weight change** — delta from previous measurement
7. **Day of week** — captures weekly behavioral patterns
8. **Days since program start** — captures long-term trends

The target variable is a composite **progress score** (0.0–1.0) that weighs workout performance (40%), nutrition adherence (35%), and goal-aligned weight trajectory (25%). The model is trained with controlled hyperparameters (`max_depth=4`, `min_samples_split=3`) to prevent overfitting on small datasets.

Feature importances from the trained model are used to determine which aspects of the user's behavior most impact their progress, enabling targeted recommendations.

---

## How It Works — User Flow

1. **Onboarding** → User fills out the profile survey → BMR/TDEE/macros are calculated → A personalized workout plan is generated → Everything is saved to Supabase.

2. **Daily Use** → User logs meals on the Nutrition page and tracks workout performance on the Workout page → Dashboard shows real-time progress with interactive charts.

3. **Weekly Adaptation** → User submits feedback on the Profile page → Clicks "Adapt My Plan" → The ML model analyzes all logged data + feedback → Recommends specific adjustments → User reviews and applies changes → A new workout plan is generated if needed.

4. **Continuous Improvement** → As more data accumulates, the ML model becomes more accurate at predicting what adjustments will drive progress → Plans evolve with the user.
