# 微信小程序开发备忘录

- view如何上下左右居中？

  wxss:

  ```css
  display: flex;
  
  justify-content: center;
  
  align-items: center;
  ```

  

- 设置 letter-spacing 后文字不能居中。

  wxss:

  ```css
  text-align: center;
  
  letter-spacing: 1em;
  
  text-indent: 1em;
  ```



- 计算屏幕宽高 （单位：rpx）

  js:

  ```javascript
  - onLoad: function (options) {
  
    ​	try {
  
    ​      wx.getSystemInfo({
  
    ​        success: function(res) {
  
    ​          that.setData({
  
    ​            windowWidth: res.windowWidth,
  
    ​            windowHeight: res.windowHeight
  
    ​          })
  
    ​        }
  
    ​      })
  
    ​    } catch(e) {
  
    ​      console.error(e)
  
    ​    }
  
  ​	}
  ```

  wxml:

  ```html
  <view class="container" style="height: {{windowHeight * 750 / windowWidth }}rpx;">
  
  </view>
  ```



- 制作只有左右有弧度、上下直线的按钮

  wxss:

  ```css
  border-top-left-radius: 40rpx;
  
  border-top-right-radius: 40rpx;
  
  border-bottom-left-radius: 40rpx;
  
  border-bottom-right-radius: 40rpx;
  ```



- 因为分享按钮要使用<button>, 定制时需要去掉按钮原来的背景样式

  wcss:

  ```css
  .share-btn {
  
  	...
  
   }
  
  
  
  .share-btn {
  
  	border: none;
  
  }
  ```

  

