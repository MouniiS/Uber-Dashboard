# NCR Ride Bookings -- Power BI Dashboard

## 1. Project overview

This project analyses ride-booking activity using a Power BI dashboard
built from the NCR ride-bookings dataset. The report is designed to help
understand booking volume, booking outcomes, vehicle performance, ride
distance, booking value, payment patterns, cancellations, and
customer/driver ratings.

The workflow has four main stages:

1.  Import the Excel dataset into Power BI.
2.  Clean and prepare the data in Power Query Editor.
3.  Build the analytical model and create DAX measures.
4.  Design interactive report pages and interpret the results.

### Project files

  -----------------------------------------------------------------------
  File                                Purpose
  ----------------------------------- -----------------------------------
  `ncr_ride_bookings.csv.xlsx`        Source dataset

  `Uber.pbix`                         Power BI report containing the
                                      dashboard and data model

  `README.md`                         Project documentation
  -----------------------------------------------------------------------

## 2. Dataset description

The source workbook contains one worksheet, `ncr_ride_bookings`, with
**150,000 rows and 21 columns**. The records cover dates from **1
January 2024 to 30 December 2024**.

### Dataset columns

  -----------------------------------------------------------------------
  Column                              Description
  ----------------------------------- -----------------------------------
  Date                                Date on which the ride was booked

  Time                                Time of the booking

  Booking ID                          Unique booking reference

  Booking Status                      Outcome/status of the booking

  Customer ID                         Identifier of the customer

  Vehicle Type                        Vehicle category selected for the
                                      ride

  Pickup Location                     Ride starting location

  Drop Location                       Ride destination

  Avg VTAT                            Average vehicle time of arrival, in
                                      minutes

  Avg CTAT                            Average customer time of arrival,
                                      in minutes

  Cancelled Rides by Customer         Indicator/count associated with
                                      customer cancellations

  Reason for cancelling by Customer   Reason recorded when the customer
                                      cancels

  Cancelled Rides by Driver           Indicator/count associated with
                                      driver cancellations

  Driver Cancellation Reason          Reason recorded when the driver
                                      cancels

  Incomplete Rides                    Indicator/count associated with
                                      incomplete rides

  Incomplete Rides Reason             Reason recorded for an incomplete
                                      ride

  Booking Value                       Monetary value associated with the
                                      booking

  Ride Distance                       Distance travelled/recorded for the
                                      ride

  Driver Ratings                      Rating given to the driver

  Customer Rating                     Customer/ride rating field in the
                                      source

  Payment Method                      Payment method used for the booking
  -----------------------------------------------------------------------

### Booking status categories

The source data contains these booking outcomes:

  Booking status                Records
  ----------------------- -------------
  Completed                      93,000
  Cancelled by Driver            27,000
  No Driver Found                10,500
  Cancelled by Customer          10,500
  Incomplete                      9,000
  **Total**                 **150,000**

These figures are a useful baseline for validating the imported data and
understanding the overall booking mix.

## 3. Importing the dataset into Power BI

1.  Open Power BI Desktop.
2.  Select **Home → Get data → Excel workbook**.
3.  Browse to `ncr_ride_bookings.csv.xlsx`.
4.  In the Navigator window, select the `ncr_ride_bookings` worksheet.
5.  Select **Transform Data**, rather than loading immediately. This
    opens Power Query Editor.
6.  Confirm that the preview shows the expected fields and records.

Power Query is the data-preparation layer. Changes made here are saved
as transformation steps and are applied again whenever the model is
refreshed.

## 4. Data cleaning and transformation in Power Query Editor

The objective of cleaning is to make the source data consistent,
correctly typed, and suitable for analysis. The following describes the
recommended cleaning workflow for the columns in this dataset. The
precise Applied Steps history is not exposed by the available report
inspection, so treat this as a reproducible explanation of the
preparation process rather than a claim that every listed operation was
definitely applied in the original PBIX.

### 4.1 Inspect the data

-   Enable **View → Column quality**, **Column distribution**, and
    **Column profile**.
-   Review valid, empty, and error values.
-   Check whether identifiers and category names contain unwanted spaces
    or quotation marks.
-   Look for inconsistent spellings and unexpected category values.
-   Check numeric columns for values stored as text.

Column profiling is particularly important because blank values can be
legitimate. For example, a cancellation reason is normally blank for a
completed ride.

### 4.2 Set appropriate data types

Use the column header data-type icon or **Transform → Data Type**.

  -----------------------------------------------------------------------
  Field(s)                            Recommended type
  ----------------------------------- -----------------------------------
  Date                                Date

  Time                                Time

  Booking ID, Customer ID             Text

  Booking Status, Vehicle Type,       Text
  locations, reasons, Payment Method  

  Avg VTAT, Avg CTAT                  Decimal number

  Cancellation/incomplete indicators  Whole number, where they represent
                                      counts or flags

  Booking Value                       Decimal number / fixed decimal as
                                      appropriate

  Ride Distance                       Decimal number

  Driver Ratings, Customer Rating     Decimal number
  -----------------------------------------------------------------------

IDs should remain text. They are labels, not quantities; converting them
to numbers can remove leading zeros or change their meaning.

### 4.3 Clean text columns

For text fields such as Booking ID, Customer ID, Booking Status, Vehicle
Type, locations, reasons, and Payment Method:

-   Apply **Transform → Format → Trim** to remove leading and trailing
    spaces.
-   Apply **Clean** to remove non-printing characters where necessary.
-   Standardise inconsistent category spelling and capitalisation.
-   If identifier values include surrounding quotation marks as literal
    characters, remove those characters consistently using Replace
    Values or an appropriate text transformation.
-   Check the result before replacing values, so valid text is not
    accidentally altered.

Consistent category labels matter because Power BI treats different
spellings as different categories in charts and slicers.

### 4.4 Handle missing values carefully

The source contains many blanks in fields that only apply to particular
booking outcomes. For example, cancellation reasons are not expected for
every completed booking, and ratings may be unavailable when a ride was
not completed.

Recommended treatment:

-   Keep a blank cancellation reason when no cancellation occurred. Do
    not automatically label it as a cancellation reason such as "Other".
-   Keep unavailable ratings blank rather than replacing them with zero.
    A zero rating would be interpreted as a real rating and could
    distort averages.
-   Keep booking value and ride distance blank where the source does not
    contain a valid value. Decide whether to exclude those records from
    a particular metric or show them separately.
-   For cancellation and incomplete-ride indicator fields, use zero only
    when the business definition confirms that blank means "not
    applicable/no event". Do not make this assumption without checking
    the source logic.
-   Remove rows only when they are demonstrably invalid or unusable, and
    record the reason for doing so.

**Important:** A blank is not always an error. It can mean "not
applicable" or "not recorded". The meaning should determine the cleaning
action.

### 4.5 Check duplicates and data validity

-   Check whether Booking ID is unique.
-   If duplicate IDs exist, investigate them before removing anything. A
    duplicate may be an accidental duplicate row or a legitimate
    repeated event, depending on the source.
-   Check that Date and Time values are valid.
-   Check that Booking Value and Ride Distance are not negative unless
    the business rules explicitly allow adjustments.
-   Check that ratings fall within the rating scale used by the source.
-   Review unexpected booking statuses and vehicle categories.

### 4.6 Create useful date attributes (if needed)

A date table or date attributes can support time-based analysis. Common
attributes include Year, Quarter, Month Number, Month Name, and Day. If
a separate calendar table is created, relate its Date column to the
booking table's Date column and sort Month Name by Month Number so
months appear chronologically.

### 4.7 Apply and load

After reviewing the transformation steps:

1.  Select **Home → Close & Apply**.
2.  Power BI applies the Power Query transformations and loads the
    prepared table into the model.
3.  Recheck row counts, data types, and key categories in Data view.

## 5. Data modeling

Data modeling defines how tables and fields are organised for
calculations and reporting.

The inspected report visuals query the `ncr_ride_bookings` table. The
dashboard therefore uses that table as its central booking fact table.
If the project is expanded, a star schema can be introduced with
separate Date, Vehicle, Customer, Location, and Payment dimensions, each
connected to the booking fact table through appropriate keys.

### Important modeling checks

-   Confirm the data type of every field.
-   Confirm that Booking ID is treated as an identifier.
-   Use a proper date field for date slicers and time trends.
-   Check that relationships are active and have the intended filter
    direction if additional tables are used.
-   Avoid many-to-many relationships unless they are required and
    understood.
-   Hide technical fields from report view when they are not useful to
    dashboard users.
-   Give measures clear names and format them appropriately.

### Why measures are used

A DAX measure calculates a result in the current filter context. For
example, a Total Bookings measure can recalculate when the user filters
the report by date or vehicle type. This makes measures more flexible
than fixed values or manually calculated totals.

## 6. DAX measures and calculations

The PBIX report contains measures/visual calculations with names
including:

-   Total Bookings
-   Cancellation Rate
-   Cancelled Bookings
-   Success Bookings
-   Success Booking Value
-   Total Booking Value
-   Total Distance Travelled
-   Avg Distance Travelled
-   Average Customer Rating
-   Average Driver Rating

The report's visual definitions confirm these names, but the exact DAX
expressions are not available in the inspected report layout. The
examples below are therefore **illustrative definitions**. Compare them
with the formulas in **Power BI Desktop → Model view/Data view → select
the measure → formula bar** before treating them as the exact formulas
used in the PBIX.

### 6.1 Total Bookings

Counts booking records with a nonblank Booking ID.

``` dax
Total Bookings =
COUNT('ncr_ride_bookings'[Booking ID])
```

This measure is used as a high-level KPI and can respond to slicers and
cross-filtering.

### 6.2 Success Bookings

Counts bookings whose status is Completed.

``` dax
Success Bookings =
CALCULATE(
    [Total Bookings],
    'ncr_ride_bookings'[Booking Status] = "Completed"
)
```

This measures completed rides, rather than all requests.

### 6.3 Cancelled Bookings

A possible definition that counts customer and driver cancellations:

``` dax
Cancelled Bookings =
CALCULATE(
    [Total Bookings],
    'ncr_ride_bookings'[Booking Status]
        IN { "Cancelled by Customer", "Cancelled by Driver" }
)
```

This definition does not include "No Driver Found" or "Incomplete". If
the project defines those outcomes as cancellations, the business rule
and formula should be adjusted.

### 6.4 Cancellation Rate

A common cancellation-rate definition is cancelled bookings divided by
all bookings.

``` dax
Cancellation Rate =
DIVIDE(
    [Cancelled Bookings],
    [Total Bookings],
    0
)
```

Format the measure as Percentage. State clearly whether the numerator
includes only customer/driver cancellations or also "No Driver Found"
and other unsuccessful outcomes.

### 6.5 Total Booking Value

``` dax
Total Booking Value =
SUM('ncr_ride_bookings'[Booking Value])
```

This adds the available booking values in the current filter context. If
the business question is completed-ride revenue, use a completed-only
measure instead.

### 6.6 Success Booking Value

``` dax
Success Booking Value =
CALCULATE(
    [Total Booking Value],
    'ncr_ride_bookings'[Booking Status] = "Completed"
)
```

This separates the value of completed bookings from the value associated
with other booking outcomes.

### 6.7 Total Distance Travelled

``` dax
Total Distance Travelled =
SUM('ncr_ride_bookings'[Ride Distance])
```

This adds the available ride-distance values. If only completed rides
should count as travelled distance, apply a Completed status filter.

### 6.8 Average Distance Travelled

``` dax
Avg Distance Travelled =
AVERAGE('ncr_ride_bookings'[Ride Distance])
```

This calculates the average of nonblank distance values. It is not the
same as total distance.

### 6.9 Average Customer Rating

``` dax
Average Customer Rating =
AVERAGE('ncr_ride_bookings'[Customer Rating])
```

### 6.10 Average Driver Rating

``` dax
Average Driver Rating =
AVERAGE('ncr_ride_bookings'[Driver Ratings])
```

Averages ignore blank values. Always consider whether the population
being rated is appropriate, for example completed rides only.

### Creating a measure in Power BI

1.  Open **Report**, **Data**, or **Model** view.
2.  Select the `ncr_ride_bookings` table in the Fields/Data pane.
3.  Select **Modeling → New measure**.
4.  Enter the DAX expression in the formula bar.
5.  Press Enter.
6.  Set the measure's format (Whole number, Decimal number, Currency, or
    Percentage).
7.  Drag the measure into a Card, chart, or table visual.
8.  Test it with slicers to confirm that it responds as expected.

## 7. Dashboard design and detailed page-by-page explanation

The PBIX report contains six report pages. The descriptions below are
based on the visual types and fields referenced by the report layout.

### Page 1 -- Landing / introduction page

This is the opening page of the report. It contains an image and an
action button, providing a starting point for navigation into the
analytical pages.

**Purpose** - Introduce the report. - Provide a clear entry point. -
Help users move into the detailed analysis.

**Design role**

A landing page separates navigation from analysis. It gives the report a
more organised, presentation-ready structure and helps users understand
that the report is divided into several analytical sections.

### Page 2 -- Overall booking analysis

This page includes a Date slicer, a Total Bookings card, a Booking
Status pie chart, and a monthly booking trend area chart.

**Visual 1: Date slicer**

The date slicer lets the user select a date or date range. The other
visuals on the page can then be analysed for the selected period,
subject to the report's interaction settings.

**Visual 2: Total Bookings card**

This KPI displays the number of booking records in the current filter
context. It gives users an immediate view of booking volume.

**Visual 3: Booking Status pie chart**

This chart divides bookings into Completed, Cancelled by Driver, No
Driver Found, Cancelled by Customer, and Incomplete. It helps compare
successful bookings with different unsuccessful outcomes.

The source-data baseline is 93,000 Completed bookings out of 150,000
records (62%). The remaining records are distributed across the other
four statuses. The pie chart should be used to see how this mix changes
under filters.

**Visual 4: Monthly booking trend area chart**

The area chart plots booking count by month. It helps identify periods
of higher and lower demand and makes changes over the year easier to
see.

**Questions this page helps answer** - How many bookings were
recorded? - What share of bookings were completed? - Which unsuccessful
booking status is most common? - How does booking volume change from
month to month? - Does the booking mix change when a date range is
selected?

### Page 3 -- Vehicle performance analysis

This page contains a table visual with vehicle-related metrics and
booking-value/distance measures.

The fields referenced by the visual include Vehicle Type, Vehicle,
Success Booking Value, Avg Distance Travelled, Total Distance Travelled,
and Total Booking Value.

**Purpose**

This page compares vehicle categories using value and distance
indicators.

**How to interpret the metrics**

-   **Vehicle Type / Vehicle:** identifies the vehicle category being
    compared.
-   **Success Booking Value:** value associated with completed bookings,
    according to the measure definition.
-   **Avg Distance Travelled:** average recorded distance for the
    selected vehicle category.
-   **Total Distance Travelled:** sum of recorded distance.
-   **Total Booking Value:** sum of available booking values, which may
    include non-completed outcomes unless the measure filters them.

**Business questions** - Which vehicle categories contribute the
greatest booking value? - Which vehicle category has the highest
completed-booking value? - Which categories have longer average rides? -
Which categories contribute most to total distance?

A fair comparison requires consistent definitions. For example, compare
completed-only value with completed-only distance if the objective is to
evaluate fulfilled rides.

### Page 4 -- Ride distance, payment and customer analysis

This page includes two clustered column charts, a customer-level table,
and a Date slicer.

**Visual 1: Ride Distance by Date Day**

The clustered column chart compares ride distance across day values. It
can help reveal differences across days of the month or recurring daily
patterns, depending on the date hierarchy used in the visual.

**Visual 2: Booking Value and Payment Method count**

The second clustered column chart uses Booking Value and a count of
Payment Method. It brings booking value and payment-method activity into
the same analysis.

When interpreting this chart, check the aggregation and axis
configuration. A sum of booking value and a count of payment-method
records are different units, so they should not be interpreted as
directly equivalent quantities.

**Visual 3: Customer booking-value table**

The table displays Customer ID and the sum of Booking Value. It helps
identify customers associated with higher recorded booking value.

**Date slicer**

The slicer supports date-based filtering of the visuals on the page.

**Business questions** - How does ride distance vary by day? - How is
booking value distributed across payment activity? - Which customers
have the highest recorded booking value? - Do customer-value patterns
change across date selections?

### Page 5 -- Cancellation analysis

This page focuses on cancellation behaviour. It contains two pie charts,
four KPI cards, and a Date slicer.

**Visual 1: Customer cancellation reasons**

This pie chart uses Reason for cancelling by Customer and the sum of
Cancelled Rides by Customer. It shows the distribution of recorded
customer cancellation reasons.

**Visual 2: Driver cancellation reasons**

This pie chart uses the driver cancellation count and Driver
Cancellation Reason. It helps identify the reasons most frequently
associated with driver cancellations.

**KPI 1: Cancellation Rate**

Shows the cancellation rate under the measure's definition and current
filter context. Confirm whether the measure includes only
customer/driver cancellations or a broader set of unsuccessful outcomes.

**KPI 2: Total Bookings**

Provides the denominator/context for understanding cancellation volume.

**KPI 3: Cancelled Bookings**

Displays the number of bookings classified as cancelled by the measure.

**KPI 4: Success Bookings**

Displays the number of completed bookings, providing a comparison with
cancellations.

**Date slicer**

Allows the user to investigate cancellation patterns for a selected
period.

**Business questions** - What are the most common customer cancellation
reasons? - What are the most common driver cancellation reasons? - How
many bookings were cancelled compared with completed bookings? - Does
the cancellation rate increase or decrease over time? - Which
cancellation reasons should operations teams investigate first?

**Business interpretation**

Cancellation analysis can help identify service issues. For example,
recurring driver-side reasons may point to operational or supply
problems, while recurring customer-side reasons may indicate
waiting-time, pricing, destination, or booking-experience issues. The
dataset's recorded reason categories should be examined before drawing a
specific conclusion.

### Page 6 -- Customer and driver rating analysis

This page contains two matrix/table-style visuals.

**Visual 1: Average Customer Rating by Vehicle**

Shows average customer rating grouped by vehicle. It helps compare
customer feedback across vehicle categories.

**Visual 2: Average Driver Rating by Vehicle**

Shows average driver rating grouped by vehicle. It helps compare
driver-rating patterns across vehicle categories.

**Business questions** - Which vehicle categories receive higher
customer ratings? - Which vehicle categories have higher driver
ratings? - Are ratings broadly consistent across vehicle categories? -
Are there vehicle categories that may need service-quality
investigation?

**Interpretation caution**

Ratings are averages of available values. If one vehicle category has
fewer rated rides than another, its average may be less representative.
Consider showing the number of ratings alongside the average when using
this analysis for operational decisions.

## 8. How interactivity works

Power BI visuals can interact through filters, slicers, and
cross-highlighting.

-   **Slicers:** selecting a date range filters the visuals configured
    to respond to that slicer.
-   **Cross-filtering:** selecting a category or segment in a chart can
    filter other visuals.
-   **Cross-highlighting:** some visuals highlight the related portion
    of another visual rather than removing the other categories.
-   **Clear selections:** use the visual's eraser/clear-selection option
    or reset the slicer to return to a broader view.
-   **Page navigation:** use the report's navigation buttons to move
    between the analytical pages.

The exact interaction behaviour depends on **Format → Edit
interactions** and the navigation-button actions configured in the PBIX.

## 9. Key observations from the source data

These are baseline facts calculated from the supplied Excel source,
before applying any additional Power Query filters:

-   The dataset contains 150,000 booking records.
-   The data spans 1 January 2024 through 30 December 2024.
-   93,000 records have status Completed (62% of all records).
-   27,000 records have status Cancelled by Driver (18%).
-   10,500 records have status No Driver Found (7%).
-   10,500 records have status Cancelled by Customer (7%).
-   9,000 records have status Incomplete (6%).

These percentages use all 150,000 source records as the denominator.
They are not necessarily identical to the dashboard's Cancellation Rate,
because that measure may use a narrower numerator and a different
business definition.

## 10. Dashboard benefits

The report brings several types of analysis into one place:

1.  **Performance monitoring:** booking volume and completion status can
    be monitored quickly.
2.  **Demand analysis:** monthly and daily trends help identify changes
    in booking activity.
3.  **Vehicle comparison:** vehicle-level distance and value measures
    support fleet analysis.
4.  **Customer analysis:** customer-level booking value can help
    identify high-value activity.
5.  **Cancellation diagnosis:** customer and driver cancellation reasons
    help identify operational issues.
6.  **Service-quality monitoring:** customer and driver ratings provide
    a view of service experience.
7.  **Interactive exploration:** date filters and visual interactions
    allow users to investigate different periods and categories without
    rebuilding charts.

## 11. Refreshing the report

When the source workbook is updated:

1.  Keep the workbook in the expected location, or update the source
    path if it has moved.
2.  Open `Uber.pbix` in Power BI Desktop.
3.  Select **Home → Refresh**.
4.  Resolve any source-path, credential, or data-type errors.
5.  Verify row counts and key measures.
6.  Review each report page to confirm that the visuals updated
    correctly.
7.  Save the PBIX file.

If the workbook's worksheet name or column names change, Power Query
steps may need to be updated.

## 12. Validation and quality checks

Before publishing or presenting the dashboard, verify:

-   Total source rows and loaded rows are understood.
-   Booking ID is not duplicated.
-   Booking Status categories match the source.
-   Date slicers cover the intended reporting period.
-   Blank values are treated according to business meaning.
-   Completed-only measures filter to Completed bookings.
-   Cancellation Rate uses a documented numerator and denominator.
-   Booking Value and Ride Distance use the intended units and
    population.
-   Average ratings use the intended rating population.
-   Monthly labels sort chronologically.
-   Slicers and navigation buttons work on all report pages.
-   Totals reconcile with a simple check in the source data.

## 13. Limitations and assumptions

-   The supplied dataset contains observational booking records; it does
    not by itself explain why a pattern occurred.
-   Blank values may be structurally expected for fields that do not
    apply to every booking status.
-   "No Driver Found", "Incomplete", and explicit cancellation statuses
    are different outcomes. They should not automatically be combined
    unless the business definition says to do so.
-   The exact Power Query Applied Steps and exact DAX formulas could not
    be read from the report layout inspection. The cleaning workflow and
    DAX snippets in this README are explanatory guidance and should be
    checked against the actual Power Query steps and measure formula bar
    in Power BI Desktop.
-   Monetary and distance units should be confirmed from the source
    documentation before presenting them as a specific currency or
    distance unit.
-   Ratings should be interpreted with the number of rated rides in
    mind.

## 14. Conclusion

The NCR Ride Bookings Power BI project transforms a large booking
dataset into an interactive report for analysing booking performance,
vehicle categories, distance, booking value, payment activity,
cancellations, and ratings.

Power Query Editor provides the data-cleaning and transformation stage.
The data model organises the fields and calculations. DAX measures
provide reusable calculations that respond to report filters. Finally,
the six dashboard pages present the results in a form that supports
exploration and business decision-making.

Together, these stages demonstrate an end-to-end Power BI analytics
workflow: **data import → data cleaning → data modeling → DAX
calculations → dashboard design → business interpretation**.

## 15. Tools and technologies

-   Microsoft Excel -- source data
-   Microsoft Power BI Desktop -- data preparation, modeling, DAX, and
    dashboard development
-   Power Query Editor -- cleaning and transformation
-   DAX (Data Analysis Expressions) -- measures and analytical
    calculations
-   Power BI visuals -- cards, pie charts, area charts, clustered column
    charts, tables, and matrix visuals
