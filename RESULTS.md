# UI lab results

Tool parts that I chose: Part 3 Robot Framework and Part 4 Cypress

## Part 0
- Step 0.1, Chrome version: Google Chrome 158.0.8127.36
- Step 0.3, ChromeDriver version: ChromeDriver 158.0.8127.36 (mac-arm64, in `~/drivers/chromedriver-mac-arm64/`), the same as Chrome
- Step 0.4, locator of Pizza, Please!: `button[name="pizza-please"]`
- Step 0.4, locator of Excluded tools: `select` (the only select element on the page)
- Step 0.4, locator of Must be vegetarian: `#default-checkbox`
- Step 0.4, locator of Custom Pizza Name: `input#pizza-name` (`#pizza-name` alone also matches the answer heading)
- Step 0.4, checked after turning on Advanced: each of the four locators found exactly 1 element; `#floating_filled` found 3, and `//label[normalize-space()='Max Calories per Slice']/preceding-sibling::input` found 1
- Step 0.5, pizza name, tool and calories of the run by hand: Lab Pizza 20261120-120733, tool Pizza cutter, 300 calories per slice (dough Thin; ingredients extra virgin olive oil, Roma tomatoes, mozzarella di bufala, mushrooms), run on 20 Nov 2026 at 12:07

| Step | Actual result | Pass or fail |
|------|---------------|--------------|
| 1    | The heading reads Looking to break out of your pizza routine? | Pass |
| 2    | Max Calories per Slice, Excluded tools, Must be vegetarian and Custom Pizza Name appeared | Pass |
| 3    | Fields show 600 (replaced the default 1000), Knife, vegetarian ticked and Lab Pizza 20261120-120733 | Pass |
| 4    | Answer shows Name: Lab Pizza 20261120-120733, tool Pizza cutter (not Knife) and 300 calories per slice (600 or fewer); it appeared a moment after the click | Pass |
| 5    | The page says Please log in first. | Pass |
| 6    | Both boxes already held admin; after Sign in the page shows Latest pizza recommendations | Pass |
| 7    | The line `Lab Pizza 20261120-120733 (tool=Pizza cutter, ingredients_number=4)` shows the same tool as in step 4; another user's pizza was at the top as (newest) | Pass |

## Part 3: Robot Framework
- robot --version: Robot Framework 7.5.1 (Python 3.13.12 on darwin)
- Step 3.2, failure message for the wrong title: Title should have been 'Web Form' but was 'Web form'.
- Step 3.8, run 1, result, elapsed time and pizza name: PASS, 9.69 s (`results/run1`), Lab Pizza 20261120-113605 (tool Scissors, 425 calories per slice)
- Step 3.8, run 2, result, elapsed time and pizza name: PASS, 9.58 s (`results/run2`), Lab Pizza 20261120-113619 (tool Pizza cutter, 500 calories per slice)

## Part 4: Cypress
- node --version, npm --version and npx cypress version: node v25.9.0, npm 11.12.1, Cypress package 16.2.1 and binary 16.2.1 (Electron 41.7.0, bundled Node 24.15.0). npm warned EBADENGINE because Cypress 16.2.1 lists Node ^22, ^24 or >=26, but it installed and verified. `npx cypress verify` failed with `bad option: --no-sandbox` until `ELECTRON_RUN_AS_NODE` (set by the VS Code terminal) was unset.
- Step 4.3, failure message for the wrong title: Timed out retrying after 10000ms, expected 'Web form' to equal 'Web Form' (- 'Web form' + 'Web Form')
- Step 4.6, tool and calories in the answer of POST /api/pizza: tool Scissors, 350 calories per slice (run 2, Lab Pizza 20261120-114222; run 1 also gave Scissors and 350). The spec checks that the page shows exactly `pizza.tool` and `calories` from the intercepted answer, so the values in the step-4 screenshot are those of the answer.
- Step 4.7, the line of the admin list that shows your pizza: `Lab Pizza 20261120-114222 (tool=Scissors, ingredients_number=5)` (run 2; another user's pizza was already above it as newest, the server is shared)
- Step 4.8, run 1, result, duration and pizza name: pizza-scenario.cy.js passed in 10 s (test 8450 ms), Lab Pizza 20261120-114008 (tool Scissors, 350 calories). web-form.cy.js, the second spec, did not run: Cypress crashed with `Error: * is not a valid URL` when it opened the second spec in Chrome 158, so the run exited 1. The same crash happens with any two specs in one run on Chrome 158, headless or headed, but not on Electron; web-form.cy.js passes when run alone (Step 4.3).
- Step 4.8, run 2, result, duration and pizza name: pizza-scenario.cy.js passed in 10 s (test 9049 ms), Lab Pizza 20261120-114222 (tool Scissors, 350 calories). web-form.cy.js crashed again with `Error: * is not a valid URL`, exit 1. `cypress/screenshots` and `cypress/videos` hold this run.
- Step 4.9 (optional), browser and result: not done

## Part B
1. The three number boxes of the home page all have the id `floating_filled`, so `$$("#floating_filled").length` returned 3 and an id locator would hit the first box, not necessarily Max Calories per Slice. The tests therefore start from the visible label text and step to its input (`//label[contains(.,'Max Calories')]/../input` in Robot, `cy.contains('label', ...).siblings('input')` in Cypress). I would give each box a unique id, for example `max-calories`, and point the label at it with `for="max-calories"`. A unique id is short, does not change when the label is reworded or translated, and does not break when the layout around the box changes, so the locator survives more page changes.
2. It is a consistency oracle: the test does not know in advance which tool is right, it only checks that two views of the same pizza agree, the customer's answer in step 4 and the admin list in step 7. It would catch a fault where the server saves a different pizza from the one it returned, for example if the admin list stored the tool of a second, regenerated pizza, or showed Knife although the customer excluded it.
3. The answer shows the ingredients, so the test could read each ingredient and check that none is meat (for example pepperoni, bacon or ham). It needs a list of which ingredients are not vegetarian, which the page does not give, so it would need an oracle from another source, such as the ingredient data of the QuickPizza API or a list agreed in the requirements. In Cypress, the intercepted answer of POST /api/pizza already holds `"vegetarian": true` and the ingredients, so the test could at least check `response.body.vegetarian` is true and compare the ingredient names with that list.
4. Robot Framework waited only where I wrote a wait keyword, such as `Wait Until Element Contains id:recommendations Name:` for the answer and `Wait Until Page Contains Element id:svelte-announcer` before typing on the admin page, each up to the 10 s library timeout, while Cypress retried every `cy.get`, `cy.contains` and `should` by itself for up to 10 s and also waited for the network answer with `cy.wait('@pizza')`. Writing QP-01 was quicker in Cypress, one 83-line spec plus a 12-line `adminLogin` command, against a 25-line test plus a 92-line resource file in Robot, but Robot needed no work outside the test, whereas Cypress cost me extra setup time: `npx cypress verify` failed with `bad option: --no-sandbox` until I unset `ELECTRON_RUN_AS_NODE`, and every multi-spec run on Chrome 158 crashed on the second spec with `Error: * is not a valid URL`. Both ran QP-01 in about the same time, Robot 9.69 s and 9.58 s and Cypress 10 s and 10 s, and both passed twice with a new pizza name each run. Robot's `log.html` was the more readable evidence, because it lists every keyword with its arguments and returned values, such as `${tool} = Scissors` and `${calories} = 425`, and shows the four screenshots at the steps that took them, while the Cypress terminal shows only pass or fail, so I had to read the tool and calories from the 10 s video and the step-4 and step-7 screenshots.
5. I would use Robot Framework for the automated UI tests of Project 2. First, in this lab it gave a clean full run every time: both `robot -d results/runN` runs passed and produced a complete report, while Cypress on Chrome 158 could not finish a run of all specs, and Project 2 asks for evidence of a full run that passes. Second, the resource file keeps the locators and steps of each page in one place and lets the test read like the written test case, with one keyword per step such as `Rate The Pizza Without Logging In`, which matches the project rule that each automated test follows the steps and expected results of its written system test case; it also runs on Python, the same language our group already uses with pytest for the API tests.
6. (Part 3) I change only the variable `${MAX CALORIES}` in `resources/quickpizza.resource`. The test file `pizza_scenario.robot` calls keywords such as `Fill In The Pizza Request` in the words of the test case and never names a locator, so it does not change at all; every test that uses the resource file works again after the one repair.
7. (Part 4) Cypress clears cookies, local storage and session storage before each test. QuickPizza keeps one browser on one of several servers with a cookie, and the admin list shows only the pizzas of that server. If steps 6 and 7 were a second `it`, that test would start without the cookie, could land on a different server, and would not find the pizza; the `pizzaName` variable and the `@tool` alias from the first test would also be gone, so step 7 would have nothing to compare.
