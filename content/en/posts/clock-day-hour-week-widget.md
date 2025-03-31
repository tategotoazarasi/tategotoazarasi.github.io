---
date: '2025-03-30T16:40:50+08:00'
draft: false
title: 'Dev Log: Adding a All-in-One Widget to Breezy Weather - The ClockDayHourWeekWidget Journey'
tags: [ "breezy-weather","android-widget","clock-day-hour-week","appwidgetprovider","remoteviews","widget-development","open-source","weather-app","kotlin","android-development","breezy-weather-widget","android-app-widget","clock-widget","hourly-forecast","daily-forecast","widget-config","remoteviews-presenter","android-layout","sharedpreferences","pendingintent" ]
---

I was tinkering with Breezy Weather, the open-source weather app, the other day. It's got a decent collection of
widgets, but I felt like something was missing – one of those "kitchen sink" widgets that just throws everything you
need onto your home screen. You know, the current time, what the weather's doing *right now*, what it's gonna do in the
*next few hours*, AND the outlook for the *next few days*. I got tired of either opening the app or juggling multiple
widgets to get the full picture. Naturally, the itch to code kicked in, and I decided to build it myself. Let's call it
the `ClockDayHourWeekWidget`.

This blog post is basically my development log. I'm jotting down the thought process, the steps I took, and a few bumps
I hit along the way. It's mainly for my future self, but hopefully, it might be useful for anyone else interested in
Android widget development or maybe even contributing to Breezy Weather. The style's going to be pretty casual – think
of it as dev notes – but I'll make sure to include the key technical bits and enough code snippets so you can understand
what's going on and potentially replicate it.

**The Goal:**

Create a new Android App Widget that displays:

1. **Current Time:** Just like your standard clock.
2. **Current Weather:** Icon, location name, current temperature.
3. **Hourly Forecast:** A glimpse of the weather (icon, time, temp) for the next few hours (e.g., the next 5).
4. **Daily Forecast:** The usual suspects (icon, day of the week, high/low temp) for the next few days (e.g., the next
   5).
5. **Configurability:** Following the Breezy Weather pattern, allow users to customize background style, transparency,
   text color, text size, clock font, etc., via a configuration screen.

Alright, goal set. Let's dive in!

## 1. The Big Picture: Standing on the Shoulders of Giants

Thankfully, Breezy Weather has a pretty well-defined structure, especially for adding new widgets. Looking at existing
files like `WidgetClockDayWeekProvider.kt` and `HourlyTrendWidgetIMP.kt`, the pattern becomes clear. To add a new
widget, you generally need these pieces:

1. **`AppWidgetProvider` (e.g., `XxxWidgetProvider.kt`):** This is the widget's entry point. It extends
   `AppWidgetProvider` and receives system broadcasts, most importantly `onUpdate`. Its main job is to kick off the real
   work of loading data and updating the view.
2. **Widget Implementation (e.g., `XxxWidgetIMP.kt`):** Often an `object` (Kotlin singleton) inheriting from
   `AbstractRemoteViewsPresenter`. This is where the magic happens: fetching data, loading user configuration, building
   the `RemoteViews` object (which defines the widget's UI), and handling click intents.
3. **Configuration Activity (e.g., `XxxWidgetConfigActivity.kt`):** An `Activity` extending
   `AbstractWidgetConfigActivity`. It pops up when the user adds the widget, allowing them to customize its appearance (
   background, colors, etc.). It also needs to show a live preview of the settings.
4. **XML Layout Files (`widget_xxx.xml`, `widget_xxx_card.xml`):** These define the static structure of the widget's UI.
   Typically, there's a version without a background card and one with it.
5. **Widget Definition XML (`xml/widget_xxx.xml`, `xml/v28/widget_xxx.xml`):** This metadata file tells the Android
   system about the widget – its minimum size, preview image, the configuration activity to launch, update frequency (
   usually 0 here, as updates are triggered programmatically), etc. The `v28` version usually adds
   `widgetFeatures="reconfigurable"`.
6. **Resource Updates:** You'll need to touch several resource files:
    * `dimens.xml`: Possibly define new dimensions if needed.
    * `keys.xml`: Add a unique SharedPreferences key for storing the widget's settings.
    * `strings.xml`: Add the user-visible name for the widget.
    * `AndroidManifest.xml`: Register the new Provider and Config Activity.
    * `Widgets.kt`: Add unique request codes for `PendingIntent`s.

Basically, follow this recipe, create or modify each part, and voilà – a new widget is born. For our
`ClockDayHourWeekWidget`, the existing `ClockDayWeekWidget` is a great starting point. It already handles the clock,
date, current weather, and daily forecast. Our main task is to surgically insert the "hourly forecast" section into it.

## 2. Getting Our Hands Dirty: Creating the Components

Let's build this thing piece by piece.

### 1. Widget Provider (`ClockDayHourWeekWidgetProvider.kt`)

This one's relatively straightforward. We can copy `WidgetClockDayWeekProvider.kt` and make a few tweaks:

* Rename the class to `ClockDayHourWeekWidgetProvider`.
* Inside the `onUpdate` method, make sure it calls the `updateWidgetView` method of our *new* implementation class,
  `ClockDayHourWeekWidgetIMP`.
* **Key Point:** When calling `weatherRepository.getWeatherByLocationId`, we absolutely *must* set both
  `withDaily = true` and `withHourly = true`. Our widget needs both sets of forecast data.

```kotlin
// src/main/java/org/breezyweather/background/receiver/widget/ClockDayHourWeekWidgetProvider.kt

package org.breezyweather.background.receiver.widget

// ... other imports ...
import org.breezyweather.remoteviews.presenters.ClockDayHourWeekWidgetIMP // Reference the new IMP
import javax.inject.Inject

@AndroidEntryPoint // Hilt annotation is crucial
class ClockDayHourWeekWidgetProvider : AppWidgetProvider() {

    @Inject lateinit var locationRepository: LocationRepository
    @Inject lateinit var weatherRepository: WeatherRepository

    @OptIn(DelicateCoroutinesApi::class) // Note: Using GlobalScope here, a common but not ideal practice in Providers
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray,
    ) {
        super.onUpdate(context, appWidgetManager, appWidgetIds)
        // Check if any widget of this type is still in use
        if (ClockDayHourWeekWidgetIMP.isInUse(context)) {
            // Launch a coroutine on the IO dispatcher to fetch data
            GlobalScope.launch(Dispatchers.IO) {
                // Get the first location (without parameters)
                val location = locationRepository.getFirstLocation(withParameters = false)
                // Call the IMP to update the view
                ClockDayHourWeekWidgetIMP.updateWidgetView(
                    context,
                    location?.copy( // Use copy to create a new object and fill in the weather
                        weather = weatherRepository.getWeatherByLocationId(
                            location.formattedId,
                            withDaily = true,  // Needed for daily data (isDaylight, daily forecast)
                            withHourly = true, // !! Must be true, we need hourly data !!
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

A quick note on `GlobalScope.launch(Dispatchers.IO)`: In the `onUpdate` method of an `AppWidgetProvider`, which runs on
the main thread and has a short lifespan, this is a fairly common way to handle potentially long-running operations like
network requests or database access. While `GlobalScope` isn't generally recommended (its coroutines are tied to the
application's lifecycle and harder to manage), it's a simpler solution in this specific context. More robust approaches
might involve `goAsync()` paired with a Hilt-injected `CoroutineScope` or even `WorkManager`, but sticking to the
existing pattern keeps things simpler here.

### 2. Widget Implementation (`ClockDayHourWeekWidgetIMP.kt`)

This is the beast. Most of the UI construction logic lives here. Again, copying `ClockDayWeekWidgetIMP.kt` gives us a
solid foundation to build upon.

**Its Main Responsibilities:**

* `updateWidgetView`: Called by the Provider. Gets the config, calls `getRemoteViews` to build the UI, and finally
  updates the widget via `AppWidgetManager`.
* `getRemoteViews`: The core method. Takes `Context`, `Location` data, and various config parameters, returning a fully
  constructed `RemoteViews` object.
* `isInUse`: Checks if any instances of this specific widget type exist.
* `setOnClickPendingIntent`: Sets up the actions (like opening the app or calendar) when users click on different parts
  of the widget.

**Breaking Down `getRemoteViews`:**

1. **Get Config & Colors:** Use `getWidgetConfig` to load saved settings and initialize `WidgetColor` to handle color
   logic based on config and day/night status.
2. **Choose Layout:** Based on `WidgetColor`'s judgment (whether to show a card background), load either
   `R.layout.widget_clock_day_hour_week` or `R.layout.widget_clock_day_hour_week_card`.
3. **Prepare Data:** Extract `weather` data from the `Location` object, get instances of `SettingsManager`,
   `ResourcesProviderFactory`, etc.
4. **Populate Sections (using `views.setXXX` methods):**
    * **Clock:** Set the `TextClock` timezone (`setTimeZone`). Control the visibility (`setViewVisibility`) of the
      different font-styled `TextClock` views based on the `clockFont` config.
    * **Date:** Set the `TextClock` timezone and date format (`setCharSequence` with `format12Hour`/`format24Hour`).
    * **Current Weather:**
        * Icon: Get the icon URI using `ResourceHelper.getWidgetNotificationIconUri` and set it with `setImageViewUri`.
          Handle potential nulls (`weather.current` or `weatherCode`) by hiding the view (
          `setViewVisibility(View.INVISIBLE)`).
        * Alternate Calendar: Set the `TextView` text based on `CalendarHelper` settings and the `hideAlternateCalendar`
          config.
        * Place & Current Temp: Concatenate the strings and set the text for the corresponding `TextView`.
    * **Hourly Forecast (The New Bit):**
        * This is the core addition. We need the `LinearLayout` container designated for the hourly forecast in our
          layout.
        * Define an array of IDs to easily access the time `TextView`, temperature `TextView`, and weather `ImageView`
          for each hourly item.
        * Get the `weather.nextHourlyForecast` list, limiting it to a maximum number (e.g., `MAX_HOURLY_ITEMS = 5`).
        * **Loop Through Data:** Iterate `min(MAX_HOURLY_ITEMS, weather.nextHourlyForecast.size)` times.
            * Get the `HourlyForecast` object for the current hour.
            * Set the time `TextView`'s text (using `hourly.date.getHour(location, context)`).
            * Set the temperature `TextView`'s text (using `temperatureUnit.getShortValueText`), handling potential
              nulls.
            * Set the weather `ImageView`'s icon (using `ResourceHelper.getWidgetNotificationIconUri`), again handling
              potential nulls for `weatherCode` and using `hourly.isDaylight` to pick the correct day/night icon.
            * **Control Visibility:** Ensure this forecast item is visible (`setVisibility(View.VISIBLE)`).
        * **Handle Excess Views:** For any placeholder views in the layout beyond the available data (e.g., layout has 5
          slots, API gives 3 hours), hide them (`setVisibility(View.GONE)`). It's best to hide the entire parent
          `LinearLayout` or `RelativeLayout` for that item.
        * **Container Visibility:** If there's *no* hourly data at all (`hourlyItemCount == 0`), hide the entire hourly
          forecast container `LinearLayout` (`widget_clock_day_hour_week_hourly_container`).

   ```kotlin
   // Inside ClockDayHourWeekWidgetIMP.kt -> getRemoteViews() (Hourly Forecast Snippet)

   // --- Hourly Forecast ---
   val hourlyIds = arrayOf(
       // ... (Define 2D array of TextView and ImageView IDs) ...
       arrayOf(R.id.widget_clock_day_hour_week_hour_time_1, R.id.widget_clock_day_hour_week_hour_temp_1, R.id.widget_clock_day_hour_week_hour_icon_1),
       // ... other hours ...
   )
   val hourlyItemCount = min(MAX_HOURLY_ITEMS, weather.nextHourlyForecast.size)
   hourlyIds.forEachIndexed { i, hourlyId ->
       if (i < hourlyItemCount) {
           val hourly = weather.nextHourlyForecast[i]
           views.setTextViewText(hourlyId[0], hourly.date.getHour(location, context)) // Set time
           views.setTextViewText(
               hourlyId[1], // Set temperature
               hourly.temperature?.temperature?.let { temperatureUnit.getShortValueText(context, it) } ?: "..."
           )
           hourly.weatherCode?.let { // Set icon
               views.setViewVisibility(hourlyId[2], View.VISIBLE)
               views.setImageViewUri(
                   hourlyId[2],
                   ResourceHelper.getWidgetNotificationIconUri(
                       provider, it, hourly.isDaylight ?: dayTime, minimalIcon, color.minimalIconColor
                   )
               )
           } ?: views.setViewVisibility(hourlyId[2], View.INVISIBLE)

           // Make sure the parent item container is visible (assuming parent ID is widget_clock_day_hour_week_hour_item_x)
           val parentId = context.resources.getIdentifier("widget_clock_day_hour_week_hour_item_${i + 1}", "id", context.packageName)
           if (parentId != 0) views.setInt(parentId, "setVisibility", View.VISIBLE)

       } else {
           // Hide unused items (preferably the parent container)
           val parentId = context.resources.getIdentifier("widget_clock_day_hour_week_hour_item_${i + 1}", "id", context.packageName)
           if (parentId != 0) views.setInt(parentId, "setVisibility", View.GONE)
           // Fallback: If parent ID isn't found, hide individual elements
           // else { views.setInt(hourlyId[0], "setVisibility", View.GONE); ... }
       }
   }
   // If no hourly data, hide the entire hourly section
   views.setViewVisibility(
       R.id.widget_clock_day_hour_week_hourly_container,
       if (hourlyItemCount > 0) View.VISIBLE else View.GONE
   )
   ```

    * **Daily Forecast:** This logic is very similar to the original `ClockDayWeekWidgetIMP`, just make sure to use the
      new IDs from our modified layout. It also needs the same treatment for handling insufficient data (hiding extra
      views) and hiding the entire daily container if no data exists.
    * **Apply Styles:**
        * Text Color: If a specific text color is configured (`textColor != Color.TRANSPARENT`), loop through *all*
          relevant `TextView`s (including the newly added hourly ones!) and use `setTextColor`.
        * Text Size: If a non-100% size is set (`textSize != 100`), calculate the `scale`, get base dimensions (
          `R.dimen.xxx`), multiply by `scale`, and then loop through *all* relevant `TextView`s, setting the size with
          `setTextViewTextSize(TypedValue.COMPLEX_UNIT_PX, size)`. Remember the new hourly `TextView`s! You might need
          different base dimensions for different parts (clock vs. content vs. hourly time vs. daily day name).
        * Clock Font: Use a `when` statement on `clockFont` to set the visibility of the appropriate `TextClock`
          container.
        * Card Background: If `color.showCard` is true, set the background drawable (`setImageViewResource`) and its
          alpha (`setInt(id, "setImageAlpha", alpha)`).
5. **Set Click Actions:** Call the `setOnClickPendingIntent` method, passing the `context`, `views`, and `location`.

**`setOnClickPendingIntent`:**

This method wires up the clickable elements (weather icon, date, clock, daily icons) to perform actions. It creates
`PendingIntent`s and binds them using `views.setOnClickPendingIntent(viewId, pendingIntent)`.

* The crucial part is giving each `PendingIntent` a **unique Request Code**. We define these constants centrally in
  `Widgets.kt`.
* Breezy Weather provides helpers for common intents:
    * `getWeatherPendingIntent`: Opens the main app screen.
    * `getDailyForecastPendingIntent`: Opens the app scrolled to the specific forecast day.
    * `getAlarmPendingIntent`: Tries to open the system alarm/clock app.
    * `getCalendarPendingIntent`: Tries to open the system calendar app.
* We need to define a *new block* of non-conflicting request codes in `Widgets.kt` for `ClockDayHourWeekWidget` (e.g.,
  starting with `14x`).

```kotlin
// Inside ClockDayHourWeekWidgetIMP.kt

private fun setOnClickPendingIntent(context: Context, views: RemoteViews, location: Location) {
    // Click main weather area -> Open App
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_weather, // ID of the main content container
        getWeatherPendingIntent(context, location, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_WEATHER) // Use new code
    )

    // Click daily forecast icon -> Open App to that day
    val todayIndex = location.weather?.todayIndex ?: 0
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_day_icon_1, // Day 1 icon ID
        getDailyForecastPendingIntent(context, location, todayIndex, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_DAILY_FORECAST_1) // New code
    )
    // ... Set similar PendingIntents for day_icon_2 to day_icon_5 ...

    // Click clock -> Open Alarm/Clock App
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_clock_light, // Light font clock ID
        getAlarmPendingIntent(context, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CLOCK_LIGHT) // New code
    )
    // ... Set similar PendingIntents for normal and black font clocks ...

    // Click date -> Open Calendar App
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_title, // Date TextClock ID
        getCalendarPendingIntent(context, Widgets.CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_CALENDAR) // New code
    )

    // Clicks for hourly forecast items could be added here if needed,
    // but the current design doesn't seem to require them.
    /*
    views.setOnClickPendingIntent(
        R.id.widget_clock_day_hour_week_hour_icon_1,
        // getHourlyForecastPendingIntent(...) // Would need a helper and codes
    )
    */
}
```

### 3. Configuration Activity (`ClockDayHourWeekWidgetConfigActivity.kt`)

This activity lets users tweak the widget when they first add it. Copying `ClockDayWeekWidgetConfigActivity.kt` is the
path of least resistance.

**Modifications Needed:**

* Rename the class to `ClockDayHourWeekWidgetConfigActivity`.
* **`initLocations()`:** Ensure `withHourly = true` when fetching weather data, just like in the Provider. Even if the
  preview doesn't show hourly details, the underlying data might be needed for other logic (like determining
  `isDaylight` accurately for icons if the current condition isn't available).
  ```kotlin
  // Inside ClockDayHourWeekWidgetConfigActivity.kt
  override suspend fun initLocations() {
      val location = locationRepository.getFirstLocation(withParameters = false)
      locationNow = location?.copy(
          weather = weatherRepository.getWeatherByLocationId(
              location.formattedId,
              withDaily = true,
              withHourly = true, // Ensure hourly data is fetched
              withMinutely = false,
              withAlerts = false
          )
      )
  }
  ```
* **`initData()`:** Set default configuration values, like the initial clock font (`clockFontValueNow`). The base class
  `AbstractWidgetConfigActivity` handles defaults for card style, color, alpha, etc.
* **`initView()`:** Control which configuration options are visible on the screen. For this widget, options for card
  style, alpha, text color, text size, clock font, and hiding the alternate calendar should all be visible.
* **`updateWidgetView()`:** When the user changes a setting in the config UI, this method calls
  `ClockDayHourWeekWidgetIMP.updateWidgetView` to immediately update the widget instance on the home screen (live
  preview effect).
* **`remoteViews` (getter):** This property provides the `RemoteViews` for the preview area within the config screen. It
  must call `ClockDayHourWeekWidgetIMP.getRemoteViews`, passing the *current* selections from the config UI (
  `cardStyleValueNow`, `cardAlpha`, `textColorValueNow`, etc.).
* **`configStoreName` (getter):** Returns the unique SharedPreferences key used to store this widget's settings. **Must
  be unique!** We'll define this key in `keys.xml`.
  ```kotlin
  // Inside ClockDayHourWeekWidgetConfigActivity.kt
  override val configStoreName: String
      get() {
          // Return the new key we define in keys.xml
          return getString(R.string.sp_widget_clock_day_hour_week_setting)
      }
  ```

### 4. XML Layout Files

We need two layout files: `layout/widget_clock_day_hour_week.xml` (no background) and
`layout/widget_clock_day_hour_week_card.xml` (with background).

Copy `widget_clock_day_week.xml` and `widget_clock_day_week_card.xml` and then modify them.

**Key Modifications:**

1. **Rename Root Layout and ALL View IDs:** To prevent clashes, systematically rename all IDs. A good practice is to
   replace `widget_clock_day_week_` with `widget_clock_day_hour_week_`.
2. **Add Hourly Forecast Section:** Between the "Date/Place/Current Temp" section and the "Daily Forecast" section,
   insert a new `LinearLayout`. Give it the ID `android:id="@+id/widget_clock_day_hour_week_hourly_container"`.
    * Set its `orientation="horizontal"`.
    * Inside it, place 5 child `LinearLayout`s (or `RelativeLayout`s), each representing one hour's forecast.
    * Set each hourly item's `LinearLayout` to `orientation="vertical"`, `layout_width="0dp"`,
      `layout_height="wrap_content"`, `layout_weight="1"`, `gravity="center_horizontal"`. Give them unique IDs like
      `widget_clock_day_hour_week_hour_item_1` through `item_5`.
    * Inside each hourly item `LinearLayout`, place the three necessary views:
        * A `TextView` for the time (`widget_clock_day_hour_week_hour_time_x`).
        * An `ImageView` for the weather icon (`widget_clock_day_hour_week_hour_icon_x`).
        * A `TextView` for the temperature (`widget_clock_day_hour_week_hour_temp_x`).
    * Use dimensions from `dimens.xml`, like `@dimen/widget_time_text_size` for the time,
      `@dimen/widget_content_text_size` for the temp, and `@dimen/widget_little_weather_icon_size` for the icon.
3. **Modify Daily Forecast IDs:** Rename the original daily forecast IDs (like `widget_clock_day_week_week_x`,
   `_temp_x`, `_icon_x`) to `widget_clock_day_hour_week_day_week_x`, `_day_temp_x`, `_day_icon_x`. Also, give the parent
   `LinearLayout` container for the daily forecast an ID, like `widget_clock_day_hour_week_daily_container`.
4. **`widget_clock_day_hour_week_card.xml`:** This file is essentially a copy of `widget_clock_day_hour_week.xml`, but
   with an `ImageView` added as the *first* child inside the root `RelativeLayout`. This `ImageView` will display the
   card background; give it the ID `widget_clock_day_hour_week_card`.

```xml
<!-- layout/widget_clock_day_hour_week.xml (Snippet showing new hourly structure) -->
<RelativeLayout ...>
<LinearLayout
android:id="@+id/widget_clock_day_hour_week_weather" ...>

		<!-- ... (Clock, Date, Current Weather sections - IDs modified) ... -->

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

<!-- Hour 2 to 5 (Similar structure) -->
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
		<!-- Day 2 to 5 (Similar structure, IDs modified) -->
		<!-- ... -->
		</LinearLayout>

		</LinearLayout>
		</RelativeLayout>
```

### 5. Widget Definition XML

Create `widget_clock_day_hour_week.xml` in `res/xml/` and a corresponding version in `res/xml-v28/` (create the
directory if it doesn't exist).

Copy `xml/widget_clock_day_week.xml` and `xml-v28/widget_clock_day_week.xml`.

**Changes to Make:**

* **`android:minWidth` / `android:minHeight`:** Since we added the hourly forecast row, the widget needs more vertical
  space. Increase `minHeight`, for example, from `@dimen/widget_grid_2` (110dp) to `@dimen/widget_grid_3` (180dp). Keep
  `minWidth` at `@dimen/widget_grid_4` (250dp).
* **`android:minResizeHeight`:** The minimum resize height also needs to increase accordingly, perhaps to
  `@dimen/widget_grid_2`.
* **`android:initialLayout`:** Point this to our new layout: `@layout/widget_clock_day_hour_week`.
* **`android:previewImage`:** Point this to a new preview drawable: `@drawable/widget_clock_day_hour_week`. **Remember,
  you need to create this image yourself and place it in the `drawable` folders.**
* **`android:configure`:** Point this to our new configuration activity:
  `org.breezyweather.remoteviews.config.ClockDayHourWeekWidgetConfigActivity`.
* **`v28` Version:** Make the same changes, and ensure `android:widgetFeatures="reconfigurable"` is present.

```xml
<!-- res/xml/widget_clock_day_hour_week.xml -->
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
                    android:minWidth="@dimen/widget_grid_4"
                    android:minHeight="@dimen/widget_grid_3" <!-- Increased height -->
		android:minResizeWidth="@dimen/widget_grid_3"
		android:minResizeHeight="@dimen/widget_grid_2" <!-- Increased resize height -->
		android:updatePeriodMillis="0"
		android:initialLayout="@layout/widget_clock_day_hour_week" <!-- Point to new layout -->
		android:previewImage="@drawable/widget_clock_day_hour_week" <!-- Point to new preview -->
		android:resizeMode="horizontal|vertical"
		android:configure="org.breezyweather.remoteviews.config.ClockDayHourWeekWidgetConfigActivity" <!-- Point to new config activity -->
		android:widgetCategory="home_screen|keyguard" />
```

## 3. Stitching It All Together: Resources & Registration

The final step is to make sure all the necessary resource definitions and registrations are in place.

1. **`dimens.xml`:** Double-check the dimensions used in the layout. Existing ones like `@dimen/widget_time_text_size` (
   10sp), `@dimen/widget_content_text_size` (14sp), `@dimen/widget_little_weather_icon_size` (36dp) seem appropriate. If
   you feel the hourly or daily sections need specific adjustments, define new dimensions here and reference them. For
   now, reusing existing ones should be fine.

2. **`keys.xml`:** Add the new `string` for the configuration storage key.
   ```xml
   <!-- res/values/keys.xml -->
   <resources ...>
       ...
       <string name="sp_widget_clock_day_hour_week_setting" translatable="false">widget_clock_day_hour_week_setting</string>
       ...
   </resources>
   ```

3. **`strings.xml`:** Add the user-visible name for the widget.
   ```xml
   <!-- res/values/strings.xml -->
   <resources ...>
       ...
       <string name="widget_clock_day_hour_week">Clock + Day + Hour + Week</string> <!-- Or your preferred name -->
       ...
   </resources>
   ```
   *(Don't forget translations in other `values-*/strings.xml` files if necessary!)*

4. **`AndroidManifest.xml`:** Inside the `<application>` tag, register the new Provider (`<receiver>`) and Config
   Activity (`<activity>`). It's good practice to group them with the other widget declarations.
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
           android:label="@string/widget_clock_day_hour_week" <!-- Reference the name from strings.xml -->
           android:exported="true">
           <meta-data
               android:name="android.appwidget.provider"
               android:resource="@xml/widget_clock_day_hour_week" /> <!-- Reference the definition xml -->
           <intent-filter>
               <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
               <action android:name="android.appwidget.action.ACTION_APPWIDGET_DISABLED" />
           </intent-filter>
       </receiver>

       ...
   </application>
   ```

5. **`Widgets.kt`:** Add the new block of PendingIntent Request Code constants. Pick an unused range (like `14x`).
   ```kotlin
   // src/main/java/org/breezyweather/remoteviews/Widgets.kt
   object Widgets {
       ... // other constants

       // clock + day + hour + week. (Using 14x block)
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
       // Add codes here if hourly forecast items become clickable
       // const val CLOCK_DAY_HOUR_WEEK_PENDING_INTENT_CODE_HOURLY_FORECAST_1 = 1471
       // ...

       ... // rest of the constants
   }
   ```

## 4. Wrapping Up & Final Thoughts

And... that should be it! After adding all these files and making the necessary resource changes, rebuild the project.
The new "Clock + Day + Hour + Week" widget should now appear in your system's widget picker. When you add it to your
home screen, the configuration activity will launch, and once configured, you should see your brand new, all-in-one
weather widget!

![](/images/68747470733a2f2f692e696d6775722e636f6d2f706379757556642e706e67.png)

**Quick Recap of the Process:**

1. **Define the Goal:** Create a comprehensive weather widget.
2. **Analyze Existing Patterns:** Identify the Provider -> IMP -> Config -> Layout -> Definition XML workflow.
3. **Copy & Modify:** Leverage existing code (`ClockDayWeek` components) as a base, then modify extensively, especially
   the IMP and Layout files.
4. **Core Addition:** Design and implement the hourly forecast section in the layout and add the corresponding
   data-binding and visibility logic in the IMP's `getRemoteViews`.
5. **Attention to Detail:** Systematically update all relevant IDs, configuration keys, widget names, and request codes
   for uniqueness. Adjust widget dimensions (`minHeight`, `minResizeHeight`).
6. **Resource Integration:** Add the necessary declarations and definitions in `AndroidManifest.xml`, `keys.xml`,
   `strings.xml`, and `Widgets.kt`.

**Potential Gotchas:**

* **`RemoteViews` Limitations:** Remember `RemoteViews` only supports a limited set of Views and methods. Complex
  interactions or custom drawing are tricky. We stuck to basics like `TextView`, `ImageView`, `LinearLayout`,
  `RelativeLayout`, and `TextClock`, which works fine.
* **ID Conflicts:** Forgetting to rename IDs after copying is an easy mistake that can lead to update errors or crashes.
  Double-check them!
* **Data Fetching:** Ensure the Provider requests `withHourly = true`, otherwise, the hourly section will be empty.
* **Layout Adaptability:** Widget appearance might need fine-tuning with `dimens.xml` values to look good across
  different screen sizes and densities.

Overall, adding the `ClockDayHourWeekWidget` was a relatively smooth process, largely thanks to Breezy Weather's clean
structure and consistent widget implementation pattern. It involved a fair amount of code, but much of it was following
the established template. The key was understanding how `RemoteViews` works and carefully handling the data binding and
view states in the `IMP` class, especially for the newly added hourly section and the visibility logic for dynamic
content.

Hope this rambling dev log is helpful to someone out there! Until the next coding adventure... Cheers!

[Source Code](https://github.com/tategotoazarasi/breezy-weather/commit/f8d8d7c258f66ead91574741322a936628bb4e7e)