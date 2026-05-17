# E2E UI Automation Framework

A **production-ready web automation framework** built with **Selenium 4.9**, **TestNG**, and **Java 17**. Implements Page Object Model (POM) design pattern with enterprise-grade reporting, retry mechanisms, and multi-browser support.

## Features

✅ **Web Automation**
- Selenium WebDriver 4.9.0 with modern locator strategies
- Cross-browser support (Chrome, Firefox, Edge, Safari)
- ThreadLocal WebDriver for parallel test execution
- Support for local and remote (Grid/Selenoid) execution

✅ **Design Patterns**
- Page Object Model (POM) for maintainability
- Base test classes with setup/teardown
- Factory pattern for driver initialization
- Fluent assertions with Hamcrest

✅ **Advanced Features**
- Automatic screenshot capture on failures
- Element highlighting for visual debugging
- Retry mechanism for flaky tests
- Custom exception handling
- Test data-driven with Excel integration

✅ **Test Reporting**
- ExtentReports 5.0.8 with HTML dashboard
- Allure Framework 2.22.2 integration
- Automatic screenshot attachment to reports
- Test execution timeline and trends

✅ **Test Listeners**
- Annotation transformer for retry logic
- Test listener for report generation
- Allure listener for rich reporting
- Custom logging and screenshots

✅ **CI/CD Integration**
- Docker containerization support
- Jenkins integration ready
- Maven parallel execution (3 fork counts)
- TestNG XML suite configuration

## Prerequisites

- **Java 17** or higher
- **Maven 3.6.0+**
- **Git**
- **Chrome/Firefox/Edge/Safari** browser (depending on test target)

Optional:
- **Docker** (for containerized execution)
- **Selenium Grid/Selenoid** (for remote execution)
- **Jenkins** (for CI/CD automation)
- **Allure Report** (for enhanced reporting)

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/robinsonmartinez23/E2E-UI-Automation-Framework.git
cd E2E-UI-Automation-Framework
```

### 2. Install Dependencies

```bash
mvn clean install
```

This will download all required dependencies:
- Selenium WebDriver 4.9.0
- TestNG 7.8.0
- ExtentReports 5.0.8
- Allure Framework 2.22.2
- Log4j 2.20.0
- Apache POI 5.2.3 (Excel utilities)

### 3. Verify Installation

```bash
mvn -version
javac -version
```

## Project Structure

```
E2E-UI-Automation-Framework/
├── src/
│   ├── main/
│   │   ├── java/com/qa/opencart/
│   │   │   ├── factory/              # DriverFactory, OptionsManager
│   │   │   ├── listeners/            # Report listeners, Retry logic
│   │   │   ├── pages/                # Page Object classes
│   │   │   │   ├── LoginPage
│   │   │   │   ├── RegisterPage
│   │   │   │   ├── AccountsPage
│   │   │   │   ├── ProductInfoPage
│   │   │   │   └── ResultsPage
│   │   │   ├── pojo/                 # Data models (User, Product)
│   │   │   ├── utils/                # AppConstants, AppErrors, Utilities
│   │   │   └── framework_exception/  # Custom exceptions
│   │   └── resources/
│   │       ├── config/               # Configuration files (*.properties)
│   │       └── testdata/             # Excel files with test data
│   │
│   └── test/
│       ├── java/com/qa/opencart/
│       │   └── tests/                # Test classes (LoginTests, RegisterTests, etc.)
│       └── resources/
│           ├── config/               # Test-specific config
│           └── testrunners/          # TestNG XML suites
│
├── pom.xml                           # Maven configuration
├── Jenkinsfile                       # Jenkins pipeline config
├── README.md                         # This file
└── .gitignore                        # Git ignore rules
```

## Configuration

### 1. Set Browser & Base URL

Edit `src/main/resources/config/application.properties`:

```properties
# Browser: chrome, firefox, edge, safari
browser=chrome

# Base URL
baseurl=https://opencart.com

# Remote execution (Grid/Selenoid)
remote=false
selenoidhost=http://localhost:4444

# Enable element highlighting for debugging
highlight=true

# Screenshot on failure
screenshot=true
```

### 2. Supported Browsers

The framework supports multiple browsers via Selenium WebDriver:

| Browser | Status | Local | Remote |
|---------|--------|-------|--------|
| Chrome  | ✅     | Yes   | Yes    |
| Firefox | ✅     | Yes   | Yes    |
| Edge    | ✅     | Yes   | Yes    |
| Safari  | ✅     | Yes   | No     |

### 3. Browser Options Configuration

Edit `OptionsManager.java` to configure browser-specific options:

```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--start-maximized");
options.addArguments("--disable-blink-features=AutomationControlled");
options.addArguments("--headless");  // Run in headless mode
```

## Running Tests

### Run All Tests

```bash
mvn test
```

### Run Specific Test Class

```bash
mvn test -Dtest=LoginTests
```

### Run Specific Test Method

```bash
mvn test -Dtest=LoginTests#testValidLogin
```

### Run with TestNG XML Suite

```bash
mvn test -Dsuite=src/main/resources/testrunners/testng_regression_selenoid.xml
```

### Run in Parallel (3 Threads)

```bash
mvn test -DforkCount=3 -DreuseForks=true
```

### Run Against Different Browser

```bash
mvn test -Dbrowser=firefox
```

### Run Against Remote Grid

```bash
mvn test -Dremote=true -Dselenoidhost=http://your-grid:4444
```

### Run in Headless Mode

```bash
mvn test -Dheadless=true
```

## Page Object Model (POM) Structure

Each page is represented as a Java class:

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage {
    private WebDriver driver;
    
    // Locators
    private By emailInput = By.name("email");
    private By passwordInput = By.name("password");
    private By loginButton = By.xpath("//button[@type='submit']");
    private By errorMessage = By.className("error");
    
    // Constructor
    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }
    
    // Page Actions
    public void enterEmail(String email) {
        driver.findElement(emailInput).sendKeys(email);
    }
    
    public void enterPassword(String password) {
        driver.findElement(passwordInput).sendKeys(password);
    }
    
    public void clickLogin() {
        driver.findElement(loginButton).click();
    }
    
    public String getErrorMessage() {
        return driver.findElement(errorMessage).getText();
    }
}
```

## Example Test Case

```java
import org.testng.annotations.Test;
import com.qa.opencart.pages.LoginPage;
import com.qa.opencart.factory.DriverFactory;

public class LoginTests extends BaseTest {
    
    @Test(priority = 1)
    public void testValidLogin() {
        LoginPage loginPage = new LoginPage(driver);
        
        loginPage.enterEmail("user@example.com");
        loginPage.enterPassword("password123");
        AccountsPage accountsPage = loginPage.clickLogin();
        
        assert accountsPage.isAccountPageDisplayed();
    }
    
    @Test(priority = 2, retryAnalyzer = Retry.class)
    public void testInvalidPassword() {
        LoginPage loginPage = new LoginPage(driver);
        
        loginPage.enterEmail("user@example.com");
        loginPage.enterPassword("wrongpassword");
        loginPage.clickLogin();
        
        String errorMsg = loginPage.getErrorMessage();
        assert errorMsg.contains("Invalid credentials");
    }
}
```

## Data-Driven Testing with Excel

### Read Test Data from Excel

```java
import com.qa.opencart.utils.ExcelUtil;

ExcelUtil excelUtil = new ExcelUtil(
    "src/main/resources/testdata/users.xlsx",
    "LoginData"
);

List<Map<String, String>> testData = excelUtil.getTestData();

testData.forEach(row -> {
    String email = row.get("email");
    String password = row.get("password");
    String expectedResult = row.get("expected");
    
    // Execute test with data
});
```

### Excel File Structure

```
| Email              | Password    | Expected Result |
|--------------------|-------------|-----------------|
| valid@test.com     | Valid123    | Success         |
| invalid@test.com   | Wrong123    | Error           |
| blank              |             | Error           |
```

## Retry Mechanism for Flaky Tests

Tests marked with `@Retry` will automatically re-run on failure:

```java
@Test(retryAnalyzer = Retry.class)
public void testFlakeyElement() {
    // Test will retry up to 2 times on failure
    // See Retry.java for configuration
}
```

Configure retry count in `Retry.java`:

```java
public int getMaxRetryCount() {
    return 2;  // Number of retries
}
```

## Screenshot Capture

Automatic screenshots are captured on test failure and attached to reports:

```java
import com.qa.opencart.factory.DriverFactory;
import org.openqa.selenium.TakesScreenshot;
import org.apache.commons.io.FileUtils;

public class ScreenshotUtil {
    
    public static String captureScreenshot(WebDriver driver, String testName) {
        File srcFile = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        String destFile = "screenshots/" + testName + ".png";
        FileUtils.copyFile(srcFile, new File(destFile));
        return destFile;
    }
}
```

## Test Reporting

### ExtentReports HTML Report

ExtentReports generates an interactive HTML report automatically:

```bash
# Reports generated in: test-output/extent-report/extent-report.html
open test-output/extent-report/extent-report.html
```

### Allure Report

Generate Allure report after test execution:

```bash
mvn test
mvn allure:report
mvn allure:serve
```

The report will open at `http://localhost:4040`

### Report Features

- Test execution timeline
- Pass/fail statistics
- Detailed logs per test
- Screenshot attachments
- Browser/OS information
- Failure trend analysis

## Docker Execution

### Build Docker Image

```bash
docker build -t ui-automation-framework:1.0 .
```

### Run Tests in Docker

```bash
docker run --rm ui-automation-framework:1.0 mvn test
```

### Run with Chrome in Docker

```bash
docker run --rm \
  --shm-size=2gb \
  ui-automation-framework:1.0 \
  mvn test -Dbrowser=chrome
```

## Jenkins CI/CD Integration

### Using Jenkinsfile

The repository includes a Jenkins pipeline configuration:

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Report') {
            steps {
                publishHTML(target: [
                    reportDir: 'test-output/extent-report',
                    reportFiles: 'extent-report.html',
                    reportName: 'ExtentReports'
                ])
            }
        }
    }
}
```

### Jenkins Parameters

```
BROWSER=chrome
BASEURL=https://staging.opencart.com
FORK_COUNT=3
REMOTE=false
```

## Troubleshooting

### Tests Running Slowly

**Solution 1: Enable Parallel Execution**
```bash
mvn test -DforkCount=3 -DreuseForks=true
```

**Solution 2: Run in Headless Mode**
```bash
mvn test -Dheadless=true
```

**Solution 3: Use Selenoid Remote Execution**
```bash
mvn test -Dremote=true
```

### Element Not Found Exception

```java
// Increase wait time
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("element")));
```

### Stale Element Reference

```java
// Refind element instead of storing reference
element = driver.findElement(By.id("dynamic"));
element.click();
```

### Chrome Driver Version Mismatch

```bash
# Selenium 4.9 manages drivers automatically via WebDriverManager
# No manual driver download needed
```

### Screenshot Directory Not Found

```bash
# Create screenshots directory
mkdir -p screenshots

# Or configure in properties
screenshot.dir=target/screenshots
```

## Best Practices

1. **Use Page Object Model** - Centralize element locators
2. **Explicit Waits** - Use WebDriverWait instead of Thread.sleep()
3. **Data Validation** - Assert both UI and data layer
4. **Meaningful Names** - Use descriptive test method names
5. **Test Independence** - Tests should not depend on each other
6. **Cleanup** - Use @AfterMethod for cleanup and report generation
7. **Screenshots on Failure** - Enable automatic screenshot capture
8. **Parameterization** - Use Excel/CSV for test data
9. **Parallel Execution** - Configure fork counts for faster runs
10. **Regular Reports** - Review Allure reports for trends

## Example Test Suite (XML)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Regression Suite" parallel="methods" thread-count="3">
    <test name="Login Tests">
        <classes>
            <class name="com.qa.opencart.tests.LoginTests"/>
        </classes>
    </test>
    
    <test name="Registration Tests">
        <classes>
            <class name="com.qa.opencart.tests.RegisterTests"/>
        </classes>
    </test>
    
    <test name="Product Tests">
        <classes>
            <class name="com.qa.opencart.tests.ProductTests"/>
        </classes>
    </test>
</suite>
```

## Common Locator Strategies

```java
// ID
By.id("element-id")

// Name
By.name("element-name")

// Class Name
By.className("element-class")

// CSS Selector
By.cssSelector("input[type='text']")

// XPath (most flexible)
By.xpath("//button[@class='login' and @type='submit']")

// Link Text
By.linkText("Click Here")

// Partial Link Text
By.partialLinkText("Click")
```

## Resources

- [Selenium 4 Documentation](https://www.selenium.dev/documentation/)
- [TestNG Official Docs](https://testng.org/doc/)
- [ExtentReports](https://www.extentreports.com/)
- [Allure Report](https://docs.qameta.io/allure/)
- [Page Object Model](https://www.selenium.dev/documentation/test_practices/encouraged/page_object_models/)

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit changes: `git commit -m 'Add new test'`
4. Push to branch: `git push origin feature/your-feature`
5. Open a Pull Request

## License

This project is provided as-is for educational and professional use.

## Author

**Robinson Martinez** - SDET & QA Automation Engineer

- LinkedIn: [linkedin.com/in/robinsonmartinezm](https://www.linkedin.com/in/robinsonmartinezm)
- GitHub: [@robinsonmartinez23](https://github.com/robinsonmartinez23)

## Support

For issues, questions, or suggestions:

1. Check the [Troubleshooting](#troubleshooting) section
2. Review test examples in `src/test/java`
3. Enable debug logging: `mvn test -X`
4. Check browser console for JavaScript errors

---

**Last Updated:** May 2026  
**Framework Version:** 1.0-SNAPSHOT
