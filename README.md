# ui-lab

ITDS362 Lab W11, Automated UI Testing. Tool parts chosen: **Part 3 Robot Framework** and **Part 4 Cypress**.
Both automate test case QP-01 (A custom pizza reaches the admin list) on https://quickpizza.grafana.com.
Results and reflection are in [RESULTS.md](RESULTS.md).

## robot/ (Part 3)
Robot Framework 7.5.1 with SeleniumLibrary 6.9.0. `first.robot` and `web_form.robot` test Selenium's web form,
`pizza_scenario.robot` runs QP-01 with keywords kept in `resources/quickpizza.resource`, and `results/run1` and
`results/run2` hold the log, report and screenshots of two full runs. Needs ChromeDriver in the `CHROMEDRIVER`
environment variable.

```bash
cd robot && python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
robot -d results/run1 -v BROWSER:headlesschrome pizza_scenario.robot
```

## cypress/ (Part 4)
Cypress 16.2.1. `cypress/e2e/web-form.cy.js` tests Selenium's web form, `cypress/e2e/pizza-scenario.cy.js` runs
QP-01 with the custom command `adminLogin` from `cypress/support/commands.js`, and `cypress/videos` and
`cypress/screenshots` hold the evidence of a full run.

```bash
cd cypress && npm install
npx cypress run --browser chrome
```
