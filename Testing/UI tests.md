User Interface (UI) tests allow us to programmatically test UI through a browser. They are simple to write and can simulate user behavior and test end-to-end integration in any environment. For this, we are using Selenium, am open source library, together with a driver, most commonly the Chrome driver.

To set up a testing project, you need to download two NuGet packages: [Selenium.WebDriver](https://www.nuget.org/packages/Selenium.WebDriver) and [Selenium.WebDriver.ChromeDriver](https://www.nuget.org/packages/Selenium.WebDriver.ChromeDriver). Note that any driver (Chrome, Mozilla, Internet Explorer, etc.) needs to be installed on the machine that is running tests and that versions of the installed browser and driver NuGet package need to match.

This is an example of a test class that uses Chrome driver to do basic UI tests.

```c#
public class InfinumTests
        : IDisposable
    {
        private IWebDriver _driver;    

    public InfinumTests()
    {
        _driver = new ChromeDriver();
        _driver.Url = "https://infinum.com";
        _driver.Manage().Window.Maximize();
    }

    [Fact]
    public void Title()
    {
        Assert.Equal("Infinum | App design & development", _driver.Title);
    }

    [Fact]
    public void Homepage_AboutUsText()
    {
        var aboutText = _driver.FindElement(By.ClassName("about-us")).Text;

        Assert.Contains("creating beautiful software", aboutText);
    }

    public void Dispose()
    {
        _driver.Quit();
    }
}
```
