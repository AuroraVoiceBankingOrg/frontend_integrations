# frontend_integrations
Frontend integrations: iOS/Android/Web/Desktop code samples, APIs for connecting UI to backend. Show how to perform banking transactions, credit card operations by voice/text. Flexible examples to adapt any future frontend tech.


# Frontend Integrations

This repository focuses on integrating various frontend platforms (mobile, web, desktop) with the AI voice assistant system. It covers UI components, communication layers, multilingual handling, and integration with banking/credit card systems. By exploring this structure and documentation, you'll gain a clear understanding of how user interactions (voice or text) flow through the frontend into the backend services, and how the frontends display results (balance queries, credit card notifications, multilingual TTS responses) returned from other parts of the system.

## High-Level Conceptual Overview

**Goal:** Provide flexible, extensible integration points for various frontends (iOS, Android, Web, Desktop) to connect the AI voice assistant backend with end-users. The frontend integrations ensure:

- Smooth voice input capture (via `voice_input_button` components).
- Language toggling (via `language_toggler`) so users can select Uzbek/Russian/English UIs.
- Secure login and token management (security login forms, auth token storage).
- Displaying dynamic data from external integrations (banking info, credit card data).
- Running locally or in the cloud, adapting to different DB backends and scaling conditions.

## Visual Flow: Interaction with Other Repositories

Imagine the system as a network of repositories, each providing a specific function. The `frontend_integrations` repo sits at the "front" of the pipeline:

```
   ┌───────────────────┐         ┌───────────────────────┐          ┌───────────────────────┐
   │   Mobile/Web/Desk │  --->   │   frontend_integrations │  --->    │   conversation_flow   │ ---> ... more repos ...
   │   UIs (Users)     │         │     repo (this)        │          │  repo                │
   └───────▲──────────┘         └───────────▲───────────┘          └───────────▲───────────┘
           │                                  │                                    │
           │ voice/text input                 │ API calls/UI updates               │
           │ and commands                     │ finalize user requests (voice or text)
           ▼                                  ▼
        (User interacts)                (Components here handle user input and send it onward)

```

- **Step-by-step**:
  1. **User speaks a query into a mobile app's voice button** (from `shared_components/voice_input_button` in this repo).
  2. The frontend code sends this captured audio/text to the `conversation_flow` repository (through APIs defined in `integration_adapters/`), triggering ASR and NLU processing in other repos (`asr_core`, `ai_nlu`, `tts_core`, `data_sources_adapters`, `external_integrations`).
  3. The backend returns processed data (like a banking balance or credit card statement) and potentially a TTS-encoded response.
  4. The `frontend_integrations` repo receives these results and updates the UI components (like `credit_card_widget` or `banking_info_display`) to show user’s balance, or triggers `tts_voice_en_expressive` in a backend repo to produce audio output if user wants voice feedback.
  5. The user sees the final results on mobile/web/desktop UI seamlessly.

**Other repositories:**

- **`config_manager`**: The frontend checks this for environment configs (dev/prod), language availability, theme overrides.
- **`asr_core`**: The frontend sends voice input to be recognized. Once recognized text is returned, the frontend shows it or uses it for next steps.
- **`tts_core`**: After backend prepares spoken responses, the frontend might fetch and play them.
- **`ai_nlu`**: The user's query is interpreted (intents/entities extracted), results come back to frontend for display.
- **`external_integrations`**: The frontend requests data from banking/credit card APIs through integration adapters here, then displays the info.
- **`utils_infra`**: The frontend might rely on logging, performance metrics, and CI/CD workflows defined here.
- **`evaluation` & `testing_envs`**: The frontend team can use scenarios from `testing_envs` to test the UI behavior under different conditions and can look at `evaluation` to understand performance metrics that might impact frontend latency displays.

## Visual Flow: Step-by-Step Running Order Within This Repo

Consider a user using a mobile app integrated with this repo:

1. **User Opens Mobile App (iOS/Android)**:  
   - The app’s main view (`mobile/ios_app/views/MainView.swift`, or `mobile/android_app/layout/main_activity_layout.xml`) loads UI components.
   - The `language_toggler` component (`shared_components/language_toggler`) initializes to the default language.
   - The `voice_input_button` (`shared_components/voice_input_button`) is active, waiting for input.
   - `banking_info_display` and `credit_card_widget` may remain hidden until data is fetched.

2. **User Presses Voice Input Button**:  
   - A voice input is captured. This triggers code in `voice_input_button` (like `VoiceInputButton.js` or `iOSVoiceInputButton.swift`) to send audio upstream.
   - The integration adapters (`integration_adapters/*`) define how to send this audio to conversation_flow and subsequently to asr_core for speech-to-text.

3. **Backend Processing**:  
   - The backend returns recognized text and possibly a banking query result (like "Your balance is 1000 USD"). The frontend receives this via `integration/banking/BankingApiAdapter.js` or `integration/credit_card/CreditCardApiAdapter.js`.
   - If text is returned that requires TTS playback, the frontend may request TTS audio from tts_core through an adapter in `integration_adapters`.

4. **UI Update**:  
   - The frontend updates `BankingInfoDisplay.js` with the fetched balance.
   - If voice output is needed, the frontend can play returned audio (fetched by `integration/frontend/mock_frontend_data.json` or direct streaming).
   - If language switching is triggered by user, `lang_mapping.yaml` in `language_support` is used to refresh UI text strings from a multilingual resource, updating all components with the chosen language strings.

5. **Security and Auth**:  
   - If user tries a protected operation (e.g., viewing detailed credit card transactions), `security_login_form/SecurityLoginForm.js` ensures user credentials or tokens are present, calling `auth_token_storage` related files to manage secure sessions.

6. **Thematic and Stylistic Adjustments**:  
   - `configs/themes` and `global_styles` directories provide CSS/SCSS for look and feel. If user toggles dark mode, `dark_theme.css` is applied. The entire UI adjusts instantly.

7. **Performance & Latency Indicators**:  
   - If latency is high (due to complex queries), `latency_indicator/LatencyIndicator.js` visually reflects it, informing user that system is working on it.
   - The developer can check performance using `web/react_app/src/hooks/useLatencyCheck.js` or `scripts/test_scenarios/run_ui_tests.sh` to ensure smooth interaction.

8. **Desktop Integration**:  
   - For desktop, `desktop/electron_app` code runs in Electron’s main and renderer processes.
   - The same components and integration adapters can be reused, ensuring consistent UX and logic across platforms.

## Detailed Explanation of File Types and Directories

- **docs/**: Architecture diagrams (`*.mmd`), user guides (`.md`), API specs (`.yaml`), and UX research notes (`.txt`) help developers understand the big picture.
- **tests/**: Unit, integration, UI, security, performance, layout tests ensure components and integrations work as intended. Developers add/modify tests before production changes.
- **configs/**: Holds `frontend_config.yaml`, theme files (`dark_theme.css`, `light_theme.css`), environment configs (dev/prod/staging). Adjust these to switch environments, languages, or UI themes without code changes.
- **global_styles/**: Contains `css`, `scss` for consistent styling. `variables.scss` defines color palettes; `responsive_rules.css` ensures UIs scale nicely on various screen sizes.
- **shared_components/**: Common UI elements like `LanguageToggler.js`, `VoiceInputButton.js`, `ErrorDisplayPanel.js`, `CreditCardWidget.js`, `BankingInfoDisplay.js`. They abstract complexity away from platform-specific code. For example, `CreditCardWidget.js` can be reused in mobile or web apps, only differing in styling or minor integration details.
- **mobile/ios_app** and **mobile/android_app**: Platform-specific directories show how to implement native views, styles, and integration on each OS. `ios_banking_adapter.swift` or `android_credit_card_api.kt` illustrate platform-specific integration code.
- **web/react_app**: Showcases a React-based interface (`ReactVoiceInputButton.jsx`, `ReactLangToggle.jsx`), hooks for managing auth tokens or latency (`useAuthToken.js`, `useLatencyCheck.js`), and a public folder with `index.html`.
- **desktop/electron_app**: Electron-based integration for desktop environments. `electron_main.js` for main process logic, `renderer_app.js` for UI rendering, both hooking into banking or credit card APIs as needed.
- **integration_adapters/**: JavaScript/TypeScript code bridging frontend calls to backend services (banking APIs, credit card APIs, multi-DB support). This allows the frontend to remain agnostic of the complex backend logic, focusing only on well-defined endpoints.
- **scripts/**: Automation for building (`build_frontend.sh`), deploying (`deploy_web.sh`), and testing scenarios (`run_ui_tests.sh`). `optimize_assets.py` can minimize image sizes or pre-compile templates for faster loading.
- **notebooks/**: `ui_experiments` includes `ui_latency_analysis.ipynb`, used by developers to experiment with UI performance, gather user feedback, or prototype new features before coding them.
- **utils/**: Hooks, CI/CD configs (`ci_pipeline.yaml`, `cd_pipeline.yaml`), logging (`log_parser.py`), and audio processing tools. If a developer wants to add precommit hooks, they go to `utils/hooks/precommit_hook.py`.
- **test_env/**: Mock and sandbox environments. For instance, `mock_mobile_frontend` simulates user queries on mobile UIs, `sandbox_banking` tests banking integration without hitting real APIs, `llm_stress_test` tries various LLM prompts. This helps ensure reliability before hitting production.

## Conclusion

By following the visual flows and explanations:

- You understand how `frontend_integrations` sits at the front of the entire system, sending user queries to backends, and displaying results.
- The repository’s structure ensures easy customization of UI elements, language toggles, secure operations, and integration with complex banking/credit card scenarios.
- Developers can quickly locate files relevant to a particular frontend or integration scenario and confidently extend or modify functionality.
- The provided scripts, configs, and test environments support continuous development, testing, and improvement of the user’s front-end experience.

In essence, `frontend_integrations` serves as a comprehensive gateway, translating user actions (voice/text) into meaningful backend queries and turning backend responses into an intuitive, multilingual, visually appealing, and secure user interface experience.
```
