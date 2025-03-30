---
date: '2025-03-30T16:40:50+08:00'
draft: false
title: '给 Breezy Weather 添加一个“全家桶”样式的新小部件：ClockDayHourWeekWidget 开发记录'
tags: [ "breezy-weather","android-widget","clock-day-hour-week","appwidgetprovider","remoteviews","widget-development","open-source","weather-app","kotlin","android-development","breezy-weather-widget","android-app-widget","clock-widget","hourly-forecast","daily-forecast","widget-config","remoteviews-presenter","android-layout","sharedpreferences","pendingintent" ]
---

最近在折腾 Breezy Weather 这个开源天气 App
的时候，发现它的小部件种类虽然不少，但好像缺了一个能把“今日信息”、“未来几小时”、“未来几天”都塞进去的“全家桶”样式。有时候就想在桌面上一次性看到所有关键信息，懒得点开
App 或者切换不同部件了。于是，手痒之下，决定自己动手，丰衣足食，给它加上这个新部件，就叫它 `ClockDayHourWeekWidget` 吧！

这篇博客主要是记录一下整个开发过程中的思考、实现步骤以及遇到的一些小坑，方便自己以后回顾，也希望能给对 Android Widget
开发或者想给 Breezy Weather 做贡献的朋友们提供一点参考。整体风格会比较随意，毕竟是写给自己的笔记嘛，但关键的技术点和代码片段会尽量给足，保证能看懂、能复现。

**最终目标：**

创建一个新的 Android App Widget，它能显示：

1. **当前时间：** 就像系统时钟那样。
2. **当前天气：** 包括天气图标、地点名称、当前温度。
3. **未来几小时天气预报：** 用小图标、时间和温度表示接下来几个小时（比如 5 个小时）的天气趋势。
4. **未来几天天气预报：** 同样用小图标、星期几和最高/最低温度展示未来几天（比如 5 天）的预报。
5. **可配置性：** 遵循 Breezy Weather 现有的模式，提供配置界面，让用户可以调整背景样式、透明度、文字颜色、大小、时钟字体等。

好，目标明确，开干！

## 一、 整体思路：站在巨人的肩膀上

Breezy Weather 的代码结构还是挺清晰的，添加新 Widget 的模式也比较固定。看了一下现有的 `WidgetClockDayWeekProvider.kt` 和
`HourlyTrendWidgetIMP.kt` 等文件，基本可以总结出添加一个新 Widget 需要搞定的几个主要部分：

1. **`AppWidgetProvider` (XXXWidgetProvider.kt):** 这是 Widget 的入口点，负责接收系统发送的更新事件 ( `onUpdate` )
   。它的主要工作就是触发真正的数据加载和视图更新逻辑。
2. **Widget 实现类 (XXXWidgetIMP.kt):** 通常是一个 `object` 单例，继承自 `AbstractRemoteViewsPresenter`
   。这是核心，负责获取数据、加载配置、构建 `RemoteViews` 对象（也就是 Widget 的界面内容），以及处理点击事件等。
3. **配置 Activity (XXXWidgetConfigActivity.kt):** 一个 `Activity`，继承自 `AbstractWidgetConfigActivity`，在用户添加
   Widget 时弹出，让用户进行个性化设置（比如背景、颜色等）。它还需要能实时预览配置效果。
4. **XML 布局文件 (`widget_xxx.xml`, `widget_xxx_card.xml`):** 定义 Widget 的静态布局结构。通常会有一个无背景版本和一个带卡片背景的版本。
5. **Widget 定义 XML (`xml/widget_xxx.xml`, `xml/v28/widget_xxx.xml`):** 向 Android 系统声明这个 Widget
   的存在，定义它的最小尺寸、预览图、配置 Activity、更新周期（虽然这里通常用 0，依靠代码触发更新）等元数据。v28 版本通常会加上
   `widgetFeatures="reconfigurable"`。
6. **资源文件更新：**
    * `dimens.xml`: 可能需要定义新的尺寸。
    * `keys.xml`: 添加用于存储 Widget 配置的 SharedPreferences Key。
    * `strings.xml`: 添加 Widget 的显示名称。
    * `AndroidManifest.xml`: 注册 Provider 和 Config Activity。
    * `Widgets.kt`: 添加用于 `PendingIntent` 的唯一请求码 (Request Code)。

基本上，只要按照这个模式，把每个部分对应创建或修改好，一个新的 Widget 就诞生了。对于 `ClockDayHourWeekWidget`，我们可以大量参考现有的
`ClockDayWeekWidget`，因为它已经包含了时钟、日期、当前天气和未来几天的功能，我们需要做的主要是在此基础上，把“未来几小时预报”这部分加进去。

## 二、 开始动手：创建各个组件

### 1. Widget Provider (`ClockDayHourWeekWidgetProvider.kt`)

这个比较简单，可以直接复制 `WidgetClockDayWeekProvider.kt`，然后做一些修改：

* 类名改成 `ClockDayHourWeekWidgetProvider`。
* 在 `onUpdate` 方法里，调用我们即将创建的 `ClockDayHourWeekWidgetIMP` 的 `updateWidgetView` 方法。
* **关键点：** 在调用 `weatherRepository.getWeatherByLocationId` 时，确保 `withDaily` 和 `withHourly` 都为 `true`。因为我们的新
  Widget 需要同时展示未来几天和未来几小时的数据。

```kotlin
// src/main/java/org/breezyweather/background/receiver/widget/ClockDayHourWeekWidgetProvider.kt

package org.breezyweather.background.receiver.widget

// ... 其他 imports ...
import org.breezyweather.remoteviews.presenters.ClockDayHourWeekWidgetIMP // 引用新的 IMP
import javax.inject.Inject

@AndroidEntryPoint // Hilt 注解不能少
class ClockDayHourWeekWidgetProvider : AppWidgetProvider() {

    @Inject lateinit var locationRepository: LocationRepository
    @Inject lateinit var weatherRepository: WeatherRepository

    @OptIn(DelicateCoroutinesApi::class) // 注意：这里用了 GlobalScope，在 Widget Provider 中这是一种常见但不完美的做法
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray,
    ) {
        super.onUpdate(context, appWidgetManager, appWidgetIds)
        // 检查这个类型的 Widget 是否还在使用
        if (ClockDayHourWeekWidgetIMP.isInUse(context)) {
            // 启动协程在 IO 线程获取数据
            GlobalScope.launch(Dispatchers.IO) {
                // 获取第一个位置信息（不带参数）
                val location = locationRepository.getFirstLocation(withParameters = false)
                // 调用 IMP 更新视图
                ClockDayHourWeekWidgetIMP.updateWidgetView(
                    context,
                    location?.copy( // 使用 copy 创建新对象并填充 weather
                        weather = weatherRepository.getWeatherByLocationId(
                            location.formattedId,
                            withDaily = true,  // 需要每日数据 (isDaylight, 每日预报)
                            withHourly = true, // !! 必须为 true，因为我们需要小时数据 !!
                            withMinutely = false,
                            withAlerts = false
                        )
                    )
                )
            }
        }
    }
}
```

这里需要注意 `GlobalScope.launch(Dispatchers.IO)` 的使用。在 `AppWidgetProvider` 的 `onUpdate`
方法中，这是一个比较常见的处理耗时操作（如网络请求、数据库查询）的方式，因为 `onUpdate` 本身运行在主线程，且生命周期短暂。虽然
`GlobalScope` 通常不被推荐（因为它创建的协程生命周期与 Application 绑定，不易管理），但在这种特定场景下，它是一个相对简单的解决方案。更好的方式可能是使用
`goAsync()` 结合 Hilt 注入的 `CoroutineScope` 或者 `WorkManager` 来处理，但为了遵循现有代码风格和简化，这里暂时保留了
`GlobalScope` 的用法。

### 2. Widget 实现类 (`ClockDayHourWeekWidgetIMP.kt`)

这是重头戏，大部分的界面构建逻辑都在这里。同样，我们可以复制 `ClockDayWeekWidgetIMP.kt` 作为基础，然后进行大量的修改和添加。

**主要职责：**

* 提供 `updateWidgetView` 方法：供 Provider 调用，负责获取配置、调用 `getRemoteViews` 构建界面、最后通过 `AppWidgetManager`
  更新 Widget。
* 提供 `getRemoteViews` 方法：这是核心，接收 `Context`、`Location` 数据和各种配置参数，返回一个构建好的 `RemoteViews` 对象。
* 提供 `isInUse` 方法：检查当前是否有此类型的 Widget 实例存在。
* 提供 `setOnClickPendingIntent` 方法：设置 Widget 上各个可点击元素的响应事件（比如点击天气区域打开 App，点击日期打开日历等）。

**`getRemoteViews` 的详细步骤拆解：**

1. **获取配置和颜色:** 使用 `getWidgetConfig` 获取用户保存的设置，并初始化 `WidgetColor` 对象来处理颜色逻辑。

2. **选择布局:** 根据 `WidgetColor` 的判断（是否显示卡片背景），选择加载 `R.layout.widget_clock_day_hour_week` 或
   `R.layout.widget_clock_day_hour_week_card`。

3. **数据准备:** 从传入的 `Location` 对象中获取 `weather` 数据，获取 `SettingsManager` 实例，准备
   `ResourcesProviderFactory` 等。

4. **填充各个区域 (使用 `views.setXXX` 系列方法):**

    * **时钟:** 设置 `TextClock` 的时区 (`setTimeZone`)，根据配置 (`clockFont`) 控制不同字体样式的 `TextClock` 的可见性 (
      `setViewVisibility`)。
    * **日期:** 设置 `TextClock` 的时区和日期格式 (`setCharSequence` 指定 `format12Hour`/`format24Hour`)。
    * **当前天气:**
        * 图标：使用 `ResourceHelper.getWidgetNotificationIconUri` 获取图标 URI，然后 `setImageViewUri`。如果
          `weather.current` 或 `weatherCode` 为空，则隐藏 (`setViewVisibility(View.INVISIBLE)`)。
        * 农历/备用日历：根据设置 (`CalendarHelper`) 和配置 (`hideAlternateCalendar`) 设置 `TextView` 的文本。
        * 地点和当前温度：拼接字符串，设置给对应的 `TextView`。
    * **小时预报 (新增部分):**
        * 这是新加的核心功能。我们需要找到布局中为小时预报准备的 `LinearLayout` 容器。
        * 定义一个 ID 数组，方便访问每个小时预报条目里的时间 `TextView`、温度 `TextView` 和天气 `ImageView`。
        * 获取 `weather.nextHourlyForecast` 列表，并限制最大显示数量（比如 5 个）。
        * **遍历数据:** 循环 `min(MAX_HOURLY_ITEMS, weather.nextHourlyForecast.size)` 次。
            * 获取对应小时的 `HourlyForecast` 对象。
            * 设置时间 `TextView` 的文本 (使用 `hourly.date.getHour(location, context)`)。
            * 设置温度 `TextView` 的文本 (使用 `temperatureUnit.getShortValueText`)，处理可能为空的情况。
            * 设置天气 `ImageView` 的图标 (使用 `ResourceHelper.getWidgetNotificationIconUri`)，同样处理 `weatherCode`
              可能为空的情况，并根据 `hourly.isDaylight` 判断使用白天还是夜晚图标。
            * **控制可见性:** 确保这个条目是可见的 (`setVisibility(View.VISIBLE)`)。
        * **处理多余的视图:** 对于超出实际数据量的预留视图（比如我们布局里放了 5 个位置，但 API 只返回了 3
          条数据），需要将它们隐藏 (`setVisibility(View.GONE)`)。最好是隐藏整个条目的父容器 `LinearLayout` 或
          `RelativeLayout`。
        * **处理容器可见性:** 如果没有任何小时数据 (`hourlyItemCount == 0`)，则隐藏整个小时预报的容器 `LinearLayout` (
          `widget_clock_day_hour_week_hourly_container`)。

   ```kotlin
   // ClockDayHourWeekWidgetIMP.kt -> getRemoteViews() 内部片段 (小时预报部分)
   
   // --- Hourly Forecast ---
   val hourlyIds = arrayOf(
       // ... (定义 TextView ID 和 ImageView ID 的二维数组) ...
       arrayOf(R.id.widget_clock_day_hour_week_hour_time_1, R.id.widget_clock_day_hour_week_hour_temp_1, R.id.widget_clock_day_hour_week_hour_icon_1),
       // ... 其他小时 ...
   )
   val hourlyItemCount = min(MAX_HOURLY_ITEMS, weather.nextHourlyForecast.size)
   hourlyIds.forEachIndexed { i, hourlyId ->
       if (i < hourlyItemCount) {
           val hourly = weather.nextHourlyForecast[i]
           views.setTextViewText(hourlyId[0], hourly.date.getHour(location, context)) // 设置时间
           views.setTextViewText(
               hourlyId[1], // 设置温度
               hourly.temperature?.temperature?.let { temperatureUnit.getShortValueText(context, it) } ?: "..."
           )
           hourly.weatherCode?.let { // 设置图标
               views.setViewVisibility(hourlyId[2], View.VISIBLE)
               views.setImageViewUri(
                   hourlyId[2],
                   ResourceHelper.getWidgetNotificationIconUri(
                       provider, it, hourly.isDaylight ?: dayTime, minimalIcon, color.minimalIconColor
                   )
               )
           } ?: views.setViewVisibility(hourlyId[2], View.INVISIBLE)
   
           // 确保整个条目的父容器可见 (假设父容器ID为 widget_clock_day_hour_week_hour_item_x)
           val parentId = context.resources.getIdentifier("widget_clock_day_hour_week_hour_item_${i + 1}", "id", context.packageName)
           if (parentId != 0) views.setInt(parentId, "setVisibility", View.VISIBLE)
   
       } else {
           // 隐藏多余的条目 (最好隐藏父容器)
           val parentId = context.resources.getIdentifier("widget_clock_day_hour_week_hour_item_${i + 1}", "id", context.packageName)
           if (parentId != 0) views.setInt(parentId, "setVisibility", View.GONE)
           // Fallback: 如果找不到父ID，隐藏单个元素
           // else { views.setInt(hourlyId[0], "setVisibility", View.GONE); ... }
       }
   }
   // 如果没有小时数据，隐藏整个小时区域
   views.setViewVisibility(
       R.id.widget_clock_day_hour_week_hourly_container,
       if (hourlyItemCount > 0) View.VISIBLE else View.GONE
   )
   ```

    * **每日预报:** 这部分逻辑与 `ClockDayWeekWidgetIMP` 基本一致，只是需要注意使用我们新布局里的
      ID。同样需要处理数据量不足时隐藏多余视图，以及没有数据时隐藏整个每日预报容器。逻辑和上面小时预报类似。
    * **应用样式:**
        * 文本颜色：如果配置了具体的文本颜色 (`textColor != Color.TRANSPARENT`)，则遍历所有 `TextView`，使用 `setTextColor`
          设置颜色。**注意要把新增的小时预报部分的 TextView 也加进来。**
        * 文本大小：如果配置了非 100% 的大小 (`textSize != 100`)，则计算缩放比例 `scale`，获取各个基础尺寸 (`R.dimen.xxx`)
          ，乘以 `scale` 得到实际尺寸，然后遍历所有 `TextView`，使用
          `setTextViewTextSize(TypedValue.COMPLEX_UNIT_PX, size)` 设置。**同样，新增的小时预报部分的 TextView 也要处理。**
          这里可能需要为不同部分的文本（如时钟、内容、小时/天的星期、小时的时间）应用不同的基础尺寸。
        * 时钟字体：使用 `when` 语句根据 `clockFont` 配置，设置对应字体 `TextClock` 容器的可见性。
        * 卡片背景：如果 `color.showCard` 为 `true`，则设置背景图 (`setImageViewResource`) 和透明度 (
          `setInt(id, "setImageAlpha", alpha)`).

5. **设置点击事件:** 调用 `setOnClickPendingIntent` 方法，传入 `context`, `views` 和 `location`。

**`setOnClickPendingIntent`:**

这个方法负责为 Widget 上的元素（如天气图标、日期、时钟、每日预报图标）设置点击后的行为。它会创建 `PendingIntent`，并使用
`views.setOnClickPendingIntent(viewId, pendingIntent)` 绑定。

* 关键在于为每个 `PendingIntent` 提供一个**唯一的 Request Code**。我们会在 `Widgets.kt` 文件中统一定义这些常量。
* Breezy Weather 提供了辅助方法来创建不同类型的 `PendingIntent`：
    * `getWeatherPendingIntent`: 点击后打开 App 主界面。
    * `getDailyForecastPendingIntent`: 点击每日预报图标后，打开 App 并滚动到对应的日期。
    * `getAlarmPendingIntent`: 点击时钟后，尝试打开系统的闹钟或时钟应用。
    * `getCalendarPendingIntent`: 点击日期后，尝试打开系统的日历应用。
* 我们需要为 `ClockDayHourWeekWidget` 在 `Widgets.kt` 中定义一套新的、不冲突的 Request Code 常量（比如使用 14x 开头）。

```kotlin
// ClockDayHourWeekWidgetIMP.kt

private fun setOnClickPendingIntent(context: Context, views: RemoteViews, location: Location) {
    // 点击天气区域 -> 打开App
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_weather, // 整个主要内容的容器 ID
        getWeatherPendingIntent(context, location, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_WEATHER) // 使用新定义的 Code
    )

    // 点击每日预报图标 -> 打开App并定位到对应天
    val todayIndex = location.weather?.todayIndex ?: 0
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_day_icon_1, // 第1天图标 ID
        getDailyForecastPendingIntent(context, location, todayIndex, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_1) // 新 Code
    )
    // ... 为 day_icon_2 到 day_icon_5 设置类似的 PendingIntent ...

    // 点击时钟 -> 打开闹钟/时钟 App
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_clock_light, // Light 字体时钟 ID
        getAlarmPendingIntent(context, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CLOCK_LIGHT) // 新 Code
    )
    // ... 为 normal 和 black 字体的时钟设置类似的 PendingIntent ...

    // 点击日期 -> 打开日历 App
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_title, // 日期 TextClock ID
        getCalendarPendingIntent(context, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CALENDAR) // 新 Code
    )

    // 如果需要，可以为小时预报的图标添加点击事件，但目前设计似乎不需要
    /*
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_hour_icon_1,
        // getHourlyForecastPendingIntent(...) // 需要定义对应的辅助方法和 Code
    )
    */
}
```

### 3. 配置 Activity (`ClockDayHourWeekWidgetConfigActivity.kt`)

这个 Activity 负责让用户在添加 Widget 时进行个性化设置。同样，复制 `ClockDayWeekWidgetConfigActivity.kt` 最省事。

**需要修改的地方：**

* 类名改为 `ClockDayHourWeekWidgetConfigActivity`。

* **`initLocations()`:** 在获取天气数据时，确保 `withHourly = true`。

  ```kotlin
  // ClockDayHourWeekWidgetConfigActivity.kt
  override suspend fun initLocations() {
      val location = locationRepository.getFirstLocation(withParameters = false)
      locationNow = location?.copy(
          weather = weatherRepository.getWeatherByLocationId(
              location.formattedId,
              withDaily = true,
              withHourly = true, // 确保获取小时数据用于可能的预览（虽然预览可能不显示小时）
              withMinutely = false,
              withAlerts = false
          )
      )
  }
  ```

* **`initData()`:** 设置默认配置值，比如时钟字体 (`clockFontValueNow`)。其他的如卡片样式、颜色、透明度等，父类
  `AbstractWidgetConfigActivity` 已经处理了。

* **`initView()`:** 控制配置界面上哪些选项可见。对于这个 Widget，卡片样式、透明度、文字颜色、文字大小、时钟字体、隐藏农历等选项都应该可见。

* **`updateWidgetView()`:** 当用户在配置界面修改选项时，调用 `ClockDayHourWeekWidgetIMP.updateWidgetView` 来触发 Widget
  实例的实时更新（预览效果）。

* **`remoteViews` (getter):** 这个属性提供一个 `RemoteViews` 对象给配置界面的预览区域。它应该调用
  `ClockDayHourWeekWidgetIMP.getRemoteViews`，传入当前的配置选项 (`cardStyleValueNow`, `cardAlpha`, `textColorValueNow`
  等)。

* **`configStoreName` (getter):** 返回用于存储这个 Widget 配置的 SharedPreferences Key。**必须是唯一的！** 我们将在
  `keys.xml` 中定义它。

  ```kotlin
  // ClockDayHourWeekWidgetConfigActivity.kt
  override val configStoreName: String
      get() {
          // 返回我们在 keys.xml 中定义的新 Key
          return getString(R.string.sp_widget_clock_day_hour_week_setting)
      }
  ```

### 4. XML 布局文件

需要创建两个布局文件：`layout/widget_clock_day_hour_week.xml` (无背景) 和 `layout/widget_clock_day_hour_week_card.xml` (
带背景)。

可以复制 `widget_clock_day_week.xml` 和 `widget_clock_day_week_card.xml`，然后进行修改。

**关键修改点：**

1. **修改根布局和所有 View 的 ID:** 为了避免冲突，最好给所有 ID 加上特定的前缀或后缀，比如把 `widget_clock_day_week_xxx`
   改成 `widget_clock_day_hour_week_xxx`。
2. **添加小时预报区域:** 在“日期/地点/当前温度”区域和“每日预报”区域之间，插入一个新的 `LinearLayout` (设置
   `android:id="@+id/widget_clock_day_hour_week_hourly_container"`)。
    * 这个 `LinearLayout` 设置为 `orientation="horizontal"`。
    * 在它内部，放置 5 个 `LinearLayout` (或 `RelativeLayout`)，每个代表一个小时的预报。
    * 每个小时的 `LinearLayout` 设置 `orientation="vertical"`, `layout_width="0dp"`, `layout_height="wrap_content"`,
      `layout_weight="1"`, `gravity="center_horizontal"`。给它们分别设置 ID，如 `widget_clock_day_hour_week_hour_item_1` 到
      `item_5`。
    * 在每个小时的 `LinearLayout` 内部，放置三个 View：
        * 一个 `TextView` 用于显示时间 (`widget_clock_day_hour_week_hour_time_x`)。
        * 一个 `ImageView` 用于显示天气图标 (`widget_clock_day_hour_week_hour_icon_x`)。
        * 一个 `TextView` 用于显示温度 (`widget_clock_day_hour_week_hour_temp_x`)。
    * 使用 `dimens.xml` 中定义的尺寸，比如 `@dimen/widget_time_text_size` 给时间，`@dimen/widget_content_text_size` 给温度，
      `@dimen/widget_little_weather_icon_size` 给图标。
3. **修改每日预报区域的 ID:** 将原有的 `widget_clock_day_week_week_x`, `_temp_x`, `_icon_x` 等 ID 修改为
   `widget_clock_day_hour_week_day_week_x`, `_day_temp_x`, `_day_icon_x`。同时，也给每日预报的父容器 `LinearLayout` 设置一个
   ID，如 `widget_clock_day_hour_week_daily_container`。
4. **`widget_clock_day_hour_week_card.xml`:** 这个文件基本就是复制 `widget_clock_day_hour_week.xml` 的内容，然后在根
   `RelativeLayout` 的最底层（第一个子 View）添加一个 `ImageView` 用于显示卡片背景，ID 设为
   `widget_clock_day_hour_week_card`。

```xml
<!-- layout/widget_clock_day_hour_week.xml (片段：展示新增的小时预报结构) -->
<RelativeLayout ...>
<LinearLayout
android:id="@+id/widget_clock_day_hour_week_weather" ...>

		<!-- ... (时钟、日期、当前天气部分，ID已修改) ... -->

		<!-- Hourly Forecast -->
<LinearLayout
android:id="@+id/widget_clock_day_hour_week_hourly_container"
android:orientation="horizontal"
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:layout_marginTop="@dimen/little_margin"
android:layout_marginBottom="@dimen/little_margin"
android:baselineAligned="false">

<!-- Hour 1 -->
<LinearLayout
		android:id="@+id/widget_clock_day_hour_week_hour_item_1"
		android:orientation="vertical"
		android:layout_width="0dp"
		android:layout_height="wrap_content"
		android:layout_weight="1"
		android:gravity="center_horizontal">

	<TextView
			android:id="@+id/widget_clock_day_hour_week_hour_time_1"
			android:textSize="@dimen/widget_time_text_size"
	... />
	<ImageView
			android:id="@+id/widget_clock_day_hour_week_hour_icon_1"
			android:layout_width="@dimen/widget_little_weather_icon_size"
			android:layout_height="@dimen/widget_little_weather_icon_size"
	... />
	<TextView
			android:id="@+id/widget_clock_day_hour_week_hour_temp_1"
			android:textSize="@dimen/widget_content_text_size"
	... />
</LinearLayout>

<!-- Hour 2 to 5 (结构类似) -->
<!-- ... -->

</LinearLayout>

		<!-- Daily Forecast -->
<LinearLayout
android:id="@+id/widget_clock_day_hour_week_daily_container"
android:orientation="horizontal"
		... >
		<!-- Day 1 -->
<LinearLayout
android:id="@+id/widget_clock_day_hour_week_day_item_1" ...>
<TextView
android:id="@+id/widget_clock_day_hour_week_day_week_1" ... />
<ImageView
android:id="@+id/widget_clock_day_hour_week_day_icon_1" ... />
<TextView
android:id="@+id/widget_clock_day_hour_week_day_temp_1" ... />
		</LinearLayout>
		<!-- Day 2 to 5 (结构类似, ID已修改) -->
		<!-- ... -->
		</LinearLayout>

		</LinearLayout>
		</RelativeLayout>
```

### 5. Widget 定义 XML

需要在 `res/xml/` 目录下创建 `widget_clock_day_hour_week.xml`，并在 `res/xml-v28/` 目录下创建同名文件（如果 v28
目录不存在，则创建它）。

可以复制 `xml/widget_clock_day_week.xml` 和 `xml-v28/widget_clock_day_week.xml`。

**修改内容：**

* **`android:minWidth` / `android:minHeight`:** 因为我们增加了小时预报，这个 Widget 需要的高度会比 `ClockDayWeek`
  更大。可以适当增加 `minHeight` 的值，比如从 `@dimen/widget_grid_2` (110dp) 增加到 `@dimen/widget_grid_3` (180dp)。
  `minWidth` 可以保持 `@dimen/widget_grid_4` (250dp)。
* **`android:minResizeHeight`:** 最小可调整高度也需要相应增加，比如增加到 `@dimen/widget_grid_2`。
* **`android:initialLayout`:** 指向我们新的布局文件 `@layout/widget_clock_day_hour_week`。
* **`android:previewImage`:** 指向一个新的预览图 `@drawable/widget_clock_day_hour_week`。*
  *这个预览图需要我们自己制作并放到 `drawable` 目录下。**
* **`android:configure`:** 指向我们新的配置 Activity
  `org.breezyweather.remoteviews.config.ClockDayHourWeekWidgetConfigActivity`。
* **`v28` 版本:** 保持修改一致，并确保 `android:widgetFeatures="reconfigurable"` 存在。

```xml
<!-- res/xml/widget_clock_day_hour_week.xml -->
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
                    android:minWidth="@dimen/widget_grid_4"
                    android:minHeight="@dimen/widget_grid_3" <!-- 增加了高度 -->
		android:minResizeWidth="@dimen/widget_grid_3"
		android:minResizeHeight="@dimen/widget_grid_2" <!-- 增加了可调整高度 -->
		android:updatePeriodMillis="0"
		android:initialLayout="@layout/widget_clock_day_hour_week" <!-- 指向新布局 -->
		android:previewImage="@drawable/widget_clock_day_hour_week" <!-- 指向新预览图 -->
		android:resizeMode="horizontal|vertical"
		android:configure="org.breezyweather.remoteviews.config.ClockDayHourWeekWidgetConfigActivity" <!-- 指向新配置 Activity -->
		android:widgetCategory="home_screen|keyguard" />
```

## 三、 整合资源与注册

最后一步，把所有需要修改或添加的资源整合起来。

1. **`dimens.xml`:** 检查一下我们布局里用到的尺寸。`@dimen/widget_time_text_size` (10sp),
   `@dimen/widget_content_text_size` (14sp), `@dimen/widget_little_weather_icon_size` (36dp)
   这些看起来都够用。如果觉得小时预报的图标或文字需要特殊大小，可以在这里添加新的 dimen 值，然后在布局里引用。目前看来，复用现有的应该问题不大。

2. **`keys.xml`:** 添加一个新的 `string` 用于存储配置。

   ```xml
   <!-- res/values/keys.xml -->
   <resources ...>
       ...
       <string name="sp_widget_clock_day_hour_week_setting" translatable="false">widget_clock_day_hour_week_setting</string>
       ...
   </resources>
   ```

3. **`strings.xml`:** 添加 Widget 的名称。

   ```xml
   <!-- res/values/strings.xml -->
   <resources ...>
       ...
       <string name="widget_clock_day_hour_week">时钟+日期+小时+星期</string> <!-- 或者其他你喜欢的名字 -->
       ...
   </resources>
   ```

   *（别忘了在其他语言的 `strings.xml` 文件中添加翻译）*

4. **`AndroidManifest.xml`:** 在 `<application>` 标签内，注册我们的 Provider (`<receiver>`) 和 Config Activity (
   `<activity>`)。建议把它们放在其他 Widget 相关声明的附近。

   ```xml
   <!-- AndroidManifest.xml -->
   <application ...>
       ...
   
       <!-- ClockDayHourWeek Widget Configuration Activity -->
       <activity
           android:name=".remoteviews.config.ClockDayHourWeekWidgetConfigActivity"
           android:theme="@style/BreezyWeatherTheme"
           android:exported="true">
           <intent-filter>
               <action android:name="android.appwidget.action.APPWIDGET_CONFIGURE" />
           </intent-filter>
       </activity>
   
       ...
   
       <!-- ClockDayHourWeek Widget Provider -->
       <receiver
           android:name=".background.receiver.widget.ClockDayHourWeekWidgetProvider"
           android:label="@string/widget_clock_day_hour_week" <!-- 引用 strings.xml 中的名称 -->
           android:exported="true">
           <meta-data
               android:name="android.appwidget.provider"
               android:resource="@xml/widget_clock_day_hour_week" /> <!-- 引用 widget 定义 xml -->
           <intent-filter>
               <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
               <action android:name="android.appwidget.action.ACTION_APPWIDGET_DISABLED" />
           </intent-filter>
       </receiver>
   
       ...
   </application>
   ```

5. **`Widgets.kt`:** 添加新的 PendingIntent Request Code 常量。找一个没被使用的数字段，比如 `14x`。

   ```kotlin
   // src/main/java/org/breezyweather/remoteviews/Widgets.kt
   object Widgets {
       ... // 其他常量
   
       // clock + day + hour + week. (使用 14x 段)
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_WEATHER = 141
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_1 = 1421
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_2 = 1422
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_3 = 1423
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_4 = 1424
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_5 = 1425
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CALENDAR = 143
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CLOCK_LIGHT = 144
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CLOCK_NORMAL = 145
       const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CLOCK_BLACK = 146
       // 如果给小时预报加了点击事件，也在这里定义 Code
       // const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_HOURLY_FORECAST_1 = 1471
       // ...
   
       ... // 其他常量
   }
   ```

## 四、 回顾与小结

好了，到这里，理论上所有需要的文件和代码修改都已经完成了。重新编译运行 App，应该就能在系统的 Widget
选择器里看到我们新增的“时钟+日期+小时+星期”小部件了。添加它到桌面时，会弹出配置界面，配置完成后，就能看到效果了！

![](/images/68747470733a2f2f692e696d6775722e636f6d2f706379757556642e706e67.png)

**整个过程回顾一下：**

1. **明确目标:** 做一个信息全面的 Widget。
2. **分析现有模式:** 找到 Provider -> IMP -> Config -> Layout -> Definition XML 的开发流程。
3. **复用与修改:** 大量复制代码 (`ClockDayWeek` 相关文件)，然后针对性修改，特别是 IMP 类和 Layout 文件。
4. **核心添加:** 在布局中加入小时预报的 `LinearLayout` 结构，并在 IMP 的 `getRemoteViews` 中添加填充该区域的逻辑，包括数据遍历和可见性控制。
5. **细节调整:** 修改所有相关的 ID、配置 Key、Widget 名称、Request Code，确保唯一性。调整 Widget 的 `minHeight` 和
   `minResizeHeight`。
6. **资源整合:** 在 `AndroidManifest.xml` 和各个资源文件 (`keys.xml`, `strings.xml`, `Widgets.kt`) 中添加必要的声明和定义。

**可能遇到的坑：**

* **`RemoteViews` 的限制:** `RemoteViews` 支持的 View 类型和方法有限，复杂交互和自定义绘制比较困难。我们这里只用了基本的
  `TextView`, `ImageView`, `LinearLayout`, `RelativeLayout` 和 `TextClock`，问题不大。
* **ID 冲突:** 如果复制粘贴时忘记修改 ID，可能会导致视图更新错误或 Crash。务必仔细检查。
* **数据获取:** 确保 Provider 里正确请求了 `withHourly = true`，否则小时数据就是空的。
* **布局适配:** 不同屏幕尺寸和密度的设备上，Widget 的显示效果可能需要微调 `dimens.xml` 中的值。

总的来说，这次添加 `ClockDayHourWeekWidget` 的过程还算顺利，主要得益于 Breezy Weather 本身良好的代码结构和清晰的 Widget
实现模式。虽然代码量不算少，但大部分是遵循既定模式的“体力活”。关键在于理解 `RemoteViews` 的工作方式，以及如何在 `IMP`
类中细心地处理数据绑定和视图状态。

希望这篇有点啰嗦的记录能帮到有需要的人！下次再折腾点别的功能，再来记录分享。