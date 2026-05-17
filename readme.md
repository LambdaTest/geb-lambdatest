# Run Geb Tests with Selenium on TestMu AI (Formerly LambdaTest)

<p align="center">
  <a href="https://www.testmuai.com/"><img src="https://img.shields.io/badge/MADE%20BY%20TestMu%20AI-000000.svg?style=for-the-badge&labelColor=000" alt="Made by TestMu AI"></a>
  <a href="https://mvnrepository.com/artifact/org.gebish/geb-core"><img src="https://img.shields.io/maven-central/v/org.gebish/geb-core.svg?style=for-the-badge&labelColor=000000" alt="Geb version"></a>
  <a href="https://community.testmuai.com/"><img src="https://img.shields.io/badge/Join%20the%20community-blueviolet.svg?style=for-the-badge&labelColor=000000" alt="Community"></a>
</p>

## Getting Started

[TestMu AI](https://www.testmuai.com/) (Formerly LambdaTest) is the world's first full-stack AI Agentic Quality Engineering platform that empowers teams to test intelligently, smarter, and ship faster. Built for scale, it offers a full-stack testing cloud with 10K+ real devices and 3,000+ browsers. With AI-native test management, MCP servers, and agent-based automation, TestMu AI supports Selenium, Appium, Playwright, and all major frameworks.

With TestMu AI (Formerly LambdaTest), you can run Geb automation tests on a cloud-based Selenium Grid of 3,000+ real browsers and operating systems. This sample shows how to configure and execute Geb tests using the TestMu AI (Formerly LambdaTest) Selenium Grid.

- [Sign up on TestMu AI](https://www.testmuai.com/register/) (Formerly LambdaTest).
- Follow the [TestMu AI documentation](https://www.testmuai.com/support/docs/) (Formerly LambdaTest) for the full setup walkthrough.

Based on Groovy, Geb is a web testing automation framework built as a wrapper over Selenium WebDriver, making it ideal for automated browser testing. Geb offers excellent features for web testing, screen scraping, and more. It queries HTML pages using jQuery-like syntax and provides built-in support for the Page Object pattern.

### Prerequisites

#### 1. Install Java For Your Operating System

* For Windows:

  Download Java for Windows from [here](http://www.java.com/en/download/manual.jsp). Run the installer and follow the setup wizard. Create a new `JAVA_HOME` environment variable pointing to the JDK folder.

* For Linux:

  ```bash
  sudo apt-get install openjdk-8-jre
  ```

* For macOS:

  Java should already be present on macOS by default.

#### 2. Maven Installation

Install Maven from [here](https://maven.apache.org/install.html).

### Setup

Clone the repository:

```bash
git clone https://github.com/LambdaTest/geb-lambdatest.git
cd geb-lambdatest
```

Add your TestMu AI (Formerly LambdaTest) `username` and `accessKey` to the `./build.gradle` in the `account{}` section.

This is the `./build.gradle` file:

```groovy
apply plugin: "idea"
apply plugin: "geb-lambdatest"

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.gebish:geb-gradle:3.3'
    }
}

allprojects {
    apply plugin: "groovy"
}

group 'org.lambdatest'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

ext {
    gebVersion = '3.3'
    seleniumVersion = '3.14.0'
}

dependencies {
    // If using Spock, need to depend on geb-spock
    testCompile "org.gebish:geb-spock:$gebVersion"
    compile "org.spockframework:spock-core:0.7-groovy-2.0"
    compile "org.seleniumhq.selenium:selenium-firefox-driver:$seleniumVersion"
    compile "org.seleniumhq.selenium:selenium-chrome-driver:$seleniumVersion"
}

repositories {
    jcenter()
}

// test task will be used when you run the tests locally, not with cloud
test {
    systemProperty "geb.build.reportsDir", reporting.file("$project.buildDir/test-results/$name/geb")
    systemProperty "geb.env", 'firefox' // select an env/browser for local testing
}

lambdaTest {
    browsers {
        firefox_windows
    }
    task {
        systemProperty "geb.build.reportsDir", reporting.file("$project.buildDir/test-results/$name/geb")
    }
    account {
        username = 'lambdatest username'      // Put Your username here
        accessKey = 'lambdatest access token' // Put Your access token here
    }
    tunnelOps {
        tunnelName = 'geb-'+UUID.randomUUID().toString()
    }
}
```

Set your credentials as environment variables.

**macOS / Linux:**

```bash
export LT_USERNAME="YOUR_USERNAME"
export LT_ACCESS_KEY="YOUR_ACCESS_KEY"
```

**Windows:**

```bash
set LT_USERNAME="YOUR_USERNAME"
set LT_ACCESS_KEY="YOUR_ACCESS_KEY"
```

## Running Geb Automation Tests On TestMu AI (Formerly LambdaTest) Selenium Grid

This is an example of incorporating the Geb TestMu AI (Formerly LambdaTest) Plugin into a Gradle build. The build is set up to work with a combination of Mozilla Firefox & Windows 10 on the TestMu AI (Formerly LambdaTest) Selenium Automation Grid.

To use it with different browser + OS combinations, alter the `src/test/resources/GebConfig.groovy` file.

This is the `GebConfig.groovy` file:

```groovy
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.firefox.FirefoxDriver
import org.openqa.selenium.remote.CapabilityType
import org.openqa.selenium.remote.DesiredCapabilities
import org.openqa.selenium.remote.RemoteWebDriver

def lambdaTestBrowser = System.getProperty("geb.lambdatest.browser")
println(lambdaTestBrowser);

if (lambdaTestBrowser) {
    driver = {
        def username = System.getenv("GEB_LAMBDATEST_USERNAME")
        assert username
        def accessKey = System.getenv("GEB_LAMBDATEST_AUTHKEY")
        assert accessKey

        final url = "https://${username}:${accessKey}@hub.lambdatest.com/wd/hub"
        println(url)
        caps = new DesiredCapabilities()
        caps.setCapability(CapabilityType.PLATFORM_NAME,"win10")
        caps.setCapability(CapabilityType.BROWSER_NAME,"firefox")
        caps.setCapability(CapabilityType.BROWSER_VERSION,"latest")
        caps.setCapability("build", "geb-lambdatest")
        def tunnelName = System.getenv("GEB_LAMBDATEST_TUNNELID")
        if(tunnelName != "null"){
            caps.setCapability("tunnel", "true")
            caps.setCapability("tunnelName", tunnelName)
        }
        println(caps)
        new RemoteWebDriver(new URL(url), caps)
    }
}

// Default browser to run on local machine
environments {

    firefox {
        driver = { new FirefoxDriver() }
    }

    chrome {
        driver = { new ChromeDriver() }
    }

}
```

A simple test that checks whether the expected title matches the page title:

This is the `LambdaTestSpec.groovy` file:

```groovy
import geb.spock.GebReportingSpec

class LambdaTestSpec extends GebReportingSpec {

    def "Open LambdaTest"() {
        when:
        to LambdaTestPage

        then:
        waitFor { at LambdaTestResultPage }

    }

}
```

This is the `LambdaTestPage.groovy` file:

```groovy
import geb.Page

class LambdaTestPage extends Page {

    static url = "https://www.lambdatest.com"
    static at = { title == "Cross Browser Testing Tools | Free Automated Website Testing | LambdaTest" }

}
```

This is the `LambdaTestResultPage.groovy` file:

```groovy
import geb.Page

class LambdaTestResultPage extends Page {
    static at = { title == "Cross Browser Testing Tools | Free Automated Website Testing | LambdaTest" }
}
```

### Run tests

For Windows:

```bash
gradlew.bat clean allLambdaTestTests --info
```

For Bash/Linux/Mac:

```bash
gradle clean allLambdaTestTests --info
```

View results on your TestMu AI dashboard.

## Testing Locally Hosted or Privately Hosted Projects

To help you perform cross browser testing of your locally stored web pages, TestMu AI (Formerly LambdaTest) provides an SSH tunnel connection called Lambda Tunnel. With Lambda Tunnel, you can test your locally hosted files before you make them live over the internet.

* Set `tunnel` value to `true` in test capabilities

OS-specific instructions to download and set up the tunnel binary:
* [Windows](https://www.testmuai.com/support/docs/display/TD/Local+Testing+For+Windows)
* [Mac](https://www.testmuai.com/support/docs/display/TD/Local+Testing+For+MacOS)
* [Linux](https://www.testmuai.com/support/docs/display/TD/Local+Testing+For+Linux)

**Important Note:** Some Safari & IE browsers don't support automatic resolution of the URL string "localhost". Therefore if you test on URLs like `http://localhost/` or `http://localhost:8080`, you would get an error in these browsers. A possible solution is to use `localhost.lambdatest.com` or replace the string "localhost" with your machine IP address.

## Contributions

Contributions are welcome. Open an issue to discuss your idea before submitting a pull request. When reporting bugs, include your Java version, OS, and Geb version.

## TestMu AI (Formerly LambdaTest) Community

Connect with testers and developers in the [TestMu AI Community](https://community.testmuai.com/). Ask questions, share what you are building, and discuss best practices in test automation and DevOps.

## TestMu AI (Formerly LambdaTest) Certifications

Earn free [TestMu AI Certifications](https://www.testmuai.com/certifications/) for testers, developers, and QA engineers. Validate your skills in Selenium, Cypress, Playwright, Appium, Espresso and more. Industry-recognized, shareable on LinkedIn, and built by practitioners, not marketers.

## Learning Resources by TestMu AI (Formerly LambdaTest)

Learn modern testing through tutorials, guides, videos, and weekly updates:

* [TestMu AI Blog](https://www.testmuai.com/blog/)
* [TestMu AI Learning Hub](https://www.testmuai.com/learning-hub/)
* [TestMu AI on YouTube](https://www.youtube.com/@TestMuAI)
* [TestMu AI Newsletter](https://www.testmuai.com/newsletter/)

## LambdaTest is Now TestMu AI

On **January 12, 2026**, [LambdaTest evolved to TestMu AI](https://www.testmuai.com/lambdatest-is-now-testmuai/), the world's first fully autonomous **Agentic AI Quality Engineering Platform**.

Same team. Same infrastructure. Same customer accounts. All existing LambdaTest logins, scripts, capabilities, and integrations continue to work without change.

Find the new home for [LambdaTest](https://www.testmuai.com).

### How LambdaTest Evolved into TestMu AI

In 2017, we launched LambdaTest with a simple mission: make testing fast, reliable, and accessible. As LambdaTest grew, we expanded into Test Intelligence, Visual Regression Testing, Accessibility Testing, API Testing, and Performance Testing, covering the full depth of the testing lifecycle.

As software development entered the AI era, testing had to evolve, too. We rebuilt the architecture to be AI-native from the ground up, with autonomous agents that **plan, author, execute, analyze, and optimize tests** while keeping humans in the loop. The platform integrates with your repos, CI, IDEs, and terminals, continuously learning from every code change and development signal.

That evolution earned a new name: **TestMu AI**, built for an AI-first future of quality engineering. TestMu is not a new name for us. It is the name of our annual community conference, which has brought together 100,000+ quality engineers to discuss how AI would reshape testing, long before that became an industry norm.

What started as a high-performance cloud testing platform has transformed into an AI-native, multi-agent system powering a connected, end-to-end quality layer. That evolution defined a new identity: LambdaTest evolved into TestMu AI, built for an AI-first future of quality engineering.

## Support

Got a question? Email [support@testmuai.com](mailto:support@testmuai.com) or chat with us 24x7 from our chat portal.
