# Image Comparison Testing for Mobile Applications with Appium

When testing mobile applications, one of the most reliable ways to ensure visual consistency across different devices and platforms is through image comparison testing. This type of testing helps identify visual differences in UI elements or entire screens, ensuring that your app's visual appearance remains intact throughout the development lifecycle.

In this article, we'll explore how to perform image comparison testing for mobile apps using Appium, a popular automation framework for mobile testing, and provide detailed insights into the methods used to capture, manipulate, and compare screenshots.

## Methods Used in Image Comparison Testing:

## 1. Centering an Element on the Screen

Before capturing a screenshot, it's often necessary to ensure that the element you want to test is visible and centered on the screen. This can be accomplished by calculating the element’s position and size and adjusting the viewport accordingly. The following method handles this by using the Appium's performActions() function, which simulates touch actions to center the element.

    async centerElementOnTheScreen(elementLocator) {
        const location = await elementLocator.getLocation();
        const size = await elementLocator.getSize();
        const centerY = location.y + size.height / 2;
    
        const windowSize = await browser.getWindowSize();
        const viewportHeight = windowSize.height;
        const viewportWidth = windowSize.width;
    
        let scrollOffsetY = centerY - viewportHeight / 2;
    
        await browser.performActions([
            {
                type: 'pointer',
                id: 'finger1',
                parameters: { pointerType: 'touch' },
                actions: [
                    { type: 'pointerMove', duration: 0, x: viewportWidth / 2, y: viewportHeight / 2 },
                    { type: 'pointerDown', button: 0 },
                    { type: 'pointerMove', duration: 2000, x: viewportWidth / 2, y: (viewportHeight / 2) - scrollOffsetY },
                    { type: 'pointerUp', button: 0 },
                ],
            },
        ]);
    }

## 2. Getting an Element’s Location and Size

To perform precise image comparison, we need the exact position and dimensions of the element being tested. This method fetches an element's location and size, which can later be used for accurate screenshot capturing.

    async getElementLocation(elementLocator) {
        const element = await $(elementLocator);
    
        const location = await element.getLocation();
        const size = await element.getSize();
    }

## 3. Taking Screenshots of the App

Taking screenshots of the app at different points in the testing process is a critical step in image comparison testing. The method below allows you to take screenshots and save them locally for comparison.

    async takeScreenshots(screenName1, screenName2, delay) {
      if (arguments.length === 3) {
          await browser.saveScreenshot(this.screenshotsPath + screenName1);
          await browser.pause(delay);
          await browser.saveScreenshot(this.screenshotsPath + screenName2);
      } else {
        console.error('Invalid number of arguments');
      }
    }

## 4. Taking a Screenshot of a Specific Element

In some cases, you may want to capture a screenshot of just a specific element on the screen instead of the entire screen. This method does just that, allowing you to capture an element's screenshot and save it for comparison.

    async takeScreenshot(screenName, elementLocator) {
      if (arguments.length === 1) {
        await browser.saveScreenshot(this.screenshotsPath + screenName);
        await this.waitForFileToExist(this.screenshotsPath + screenName);
        await this.resizeScreenshot(screenName, "temp.png", 200, 300);
      } else if (arguments.length === 2) {
        const element = await elementLocator;
        element.saveScreenshot(this.screenshotsPath + screenName);
        await this.waitForFileToExist(this.screenshotsPath + screenName);
        await this.resizeScreenshot(screenName, "temp.png", 200, 300);
      } else {
        console.error('Invalid number of arguments');
      }
    }

## 5. Resizing the Screenshot for Comparison

Sometimes, for better comparison accuracy or efficiency, it's necessary to resize the screenshots before comparing them. This method resizes the screenshot to a given width and height.

    async resizeScreenshot(screenName, screenName2, width, height) {
      await sharp(this.screenshotsPath + screenName).resize(width, height).toFile(this.screenshotsPath + screenName2);
      await fs.promises.unlink(this.screenshotsPath + screenName);
      await fs.promises.rename(this.screenshotsPath + screenName2, this.screenshotsPath + screenName);
    }

## 6. Comparing Screenshots for Visual Differences

The core of image comparison testing lies in comparing two screenshots to detect differences. The method below uses pixelmatch, a library for comparing images by checking pixel differences. It saves the diff image and logs the number of differing pixels.

    async compareScreenshots(screenName1, screenName2, sign, thresholdValue1, thresholdValue2 = 0) {
      const img1 = PNG.sync.read(fs.readFileSync(this.screenshotsPath + screenName1));
      const img2 = PNG.sync.read(fs.readFileSync(this.screenshotsPath + screenName2));

      const { width, height } = img1;
      const diff = new PNG({ width, height });

      const numDiffPixels = pixelmatch(img1.data, img2.data, diff.data, width, height, { threshold: 0.1 });

      fs.writeFileSync(this.screenshotsPath + 'diff.png', PNG.sync.write(diff));
      console.log("differences: ", numDiffPixels);

      switch (sign) {
          case 'moreThan':
              if (numDiffPixels < thresholdValue1) {
                  console.log('Image comparison failed: Too little differences.');
              } else {
                  console.log('Image comparison passed: Acceptable differences.');
              }
              expect(numDiffPixels).toBeGreaterThanOrEqual(thresholdValue1);
              break;
          case 'lessThan':
              if (numDiffPixels > thresholdValue1) {
                  console.log('Image comparison failed: Too many differences.');
              } else {
                  console.log('Image comparison passed: Acceptable differences.');
              }
              expect(numDiffPixels).toBeLessThan(thresholdValue1);
              break;
          case 'between':
              if (numDiffPixels > thresholdValue2 || numDiffPixels < thresholdValue) {
                  console.log('Image comparison failed: Outside the acceptable range.');
              } else {
                  console.log('Image comparison passed: Acceptable differences.');
              }
              expect(numDiffPixels).toBeGreaterThanOrEqual(thresholdValue1);
              expect(numDiffPixels).toBeLessThanOrEqual(thresholdValue2);
              break;
      }
    }

## Conclusion

In mobile app testing, image comparison testing is a valuable technique for verifying the visual consistency of your app. By using Appium’s capabilities for interacting with elements, capturing screenshots, and comparing visual differences, you can ensure that your app's user interface remains consistent across different devices and environments.

By automating this process, you can significantly reduce the time and effort required for manual visual testing and increase the reliability of your tests. With tools like pixelmatch and Allure for reporting, you can generate precise and easily interpretable reports that help maintain a high-quality user experience.
