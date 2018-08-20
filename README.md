1) Create Angular App
    ```sh
    ng new AngularCucumber
    ```

1) Install the global tools:
    ```sh
    npm install -g protractor cucumber
    ```
1) Install the required libraries:
    ```sh
    npm install --save-dev cucumber @types/cucumber protractor-cucumber-framework chai @types/chai chai-as-promised @types/chai globule mkdirp
    ```

1) Replace the content of `e2e/protractor.conf.js` with:
    ```js
    // Protractor configuration file, see link for more information
    // https://github.com/angular/protractor/blob/master/lib/config.ts

    exports.config = {
        allScriptsTimeout: 11000,
        specs: [
            './src/features/**/*.feature'
        ],
        capabilities: {
            'browserName': 'chrome'
        },
        directConnect: true,
        baseUrl: 'http://localhost:4200/',
        framework: 'custom',
        frameworkPath: require.resolve('protractor-cucumber-framework'),
        cucumberOpts: {
            compiler: ['ts:ts-node/register'],
            require: [ './src/features/**/*.e2e-spec.ts' ],
            tags: ["~@ignore"],
            strict: true,
            format: 'json:./TestResults/integrationtest/testresults/json/cucumber_report.json',
            'dry-run': false
        },
        suites: getSuites(),
        onPrepare() {
            require('ts-node').register({
                project: require('path').join(__dirname, './tsconfig.e2e.json')
            });
            const dirName = process.cwd() + '/TestResults/integrationtest/testresults/json';
            if (!require('fs').existsSync(dirName)) {
                require('mkdirp').sync(dirName);
            }
        }
    };

    function getSuites() {
        const fileSufix = ".feature";
        const globulePathsArray = require(`globule`).find([ `${process.cwd()}/e2e/src/features/**/*${fileSufix}` ]);
        const suites = { };
        globulePathsArray.forEach(pathFile => {
            const filename = require("path").basename(pathFile);
            const filenameWithoutSufix = filename.replace(fileSufix, '');
            suites[filenameWithoutSufix] = pathFile;
        });
        return suites;
    }
    ```

1) Copy the files of `e2e/src` to `e2e/src/pages`
1) Create the directory `e2e/src/features`
1) Create the file `e2e/src/features/example.feature` with the content:
    ```gherkin
    Feature: Home

    Scenario: A first scenario
        Given I am on the software
        When i go to the app page
        Then should display the welcome message "Welcome to AngularCucumber!"
    ```
1) Create the file `e2e/src/features/example/example.e2e-spec.ts` with the content:
    ```typescript
    import { Given, When, Then } from 'cucumber';
    import * as chai from 'chai';
    import * as chaiAsPromised from 'chai-as-promised';
    import { AppPage } from '../../pages/app.po';
    chai.use(chaiAsPromised);
    const expect = chai.expect;


    let page: AppPage;

    Given(/^I am on the software$/, () => {
        page = new AppPage();
    });

    When(/^i go to the app page$/, () => {
        return page.navigateTo();
    });

    Then(/^should display the welcome message "(.*?)"$/, async (expectedTitle: string) => {
        return page.getParagraphText().then(paragraphText => expect(paragraphText).to.be.equal(expectedTitle));
    });
    ```
1) Execute `ng e2e --suite=example`
