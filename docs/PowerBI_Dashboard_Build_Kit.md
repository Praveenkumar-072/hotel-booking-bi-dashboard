# Power BI build kit: Hotel booking dashboard

## 1. Model (Step 4)
Relationships (many to one, single direction):
- FactBookings[hotel_id] -> DimHotel[hotel_id]
- FactBookings[room_type_reserved] -> DimRoomType[room_type]
- FactBookings[country] -> DimCountry[country_code]
- FactBookings[arrival_date] -> DimDate[Date]

Date table (Modeling > New table), then Table tools > Mark as date table:
```DAX
DimDate =
ADDCOLUMNS(
    CALENDAR(DATE(2022,1,1), DATE(2024,12,31)),
    "Year", YEAR([Date]),
    "Month No", MONTH([Date]),
    "Month", FORMAT([Date], "MMM")
)
```
Select DimDate[Month] > Column tools > Sort by column > Month No.

## 2. Measures (Modeling > New measure)
```DAX
Total Bookings = COUNTROWS(FactBookings)
Cancelled Bookings = CALCULATE([Total Bookings], FactBookings[is_canceled] = 1)
Cancellation Rate = DIVIDE([Cancelled Bookings], [Total Bookings])
Total Revenue = SUM(FactBookings[Booking Revenue])
Revenue Lost = SUM(FactBookings[Lost Revenue])
Avg ADR = CALCULATE(AVERAGE(FactBookings[adr_inr]), FactBookings[is_canceled] = 0)
Room Nights Sold = CALCULATE(SUM(FactBookings[Total Nights]), FactBookings[is_canceled] = 0)
Rooms Available =
SUMX(
    VALUES(DimHotel[hotel_id]),
    CALCULATE(MAX(DimHotel[total_rooms])) * COUNTROWS(DimDate)
)
Occupancy % = DIVIDE([Room Nights Sold], [Rooms Available])
RevPAR = DIVIDE([Total Revenue], [Rooms Available])
Repeat Guest % = DIVIDE(CALCULATE([Total Bookings], FactBookings[is_repeated_guest] = 1), [Total Bookings])
Revenue MoM % =
VAR Prev = CALCULATE([Total Revenue], DATEADD(DimDate[Date], -1, MONTH))
RETURN DIVIDE([Total Revenue] - Prev, Prev)
```
Checks with no filters: Total Bookings 34,407; Cancellation Rate 30.8%; Total Revenue 382,878,893; Avg ADR about 5,980.

## 3. Page layout (one page, 1280 x 720, turn on View > Page view > Fit to page)
Theme: background #EDF3F2, cards white, teal #1F7A7A, brass #B8892B, rose #B83A4B, font Segoe UI.

| Zone | Visual | Fields |
|---|---|---|
| Top-left | Slicer (tile style) | DimDate[Year] |
| Top-right | Slicer (tile style) | DimHotel[hotel_name] |
| Row 1 | 6 Card visuals | Total Bookings, Total Revenue, Cancellation Rate, Avg ADR, Occupancy %, Revenue Lost (rose colour) |
| Row 2, full width | Matrix, conditional formatting | Rows: DimDate[Year]; Columns: DimDate[Month]; Values: Occupancy %. Cell elements > Background colour > gradient, light teal to teal |
| Row 3, left (wide) | Line and clustered column chart | X: DimDate[Month]; Column: Total Revenue (brass); Line: Cancellation Rate (rose) |
| Row 3, right | Clustered bar | Axis: Lead Time Bucket; Value: Cancellation Rate |
| Row 4, left | Clustered bar | Axis: market_segment; Value: Cancellation Rate |
| Row 4, middle | Clustered bar | Axis: deposit_type; Value: Cancellation Rate |
| Row 4, right | Filled map or bar chart | DimCountry[country_name]; Value: Total Revenue |

Sort Lead Time Bucket: add a helper column in Power Query (0-7 = 1, 8-30 = 2, 31-90 = 3, 90+ = 4), then Sort by column.

## 4. Make it responsive
1. View > Mobile layout, and drag the cards, slicers and 3 main charts into the phone canvas.
2. Format page > Canvas settings > Type: 16:9, or use Fit to width for wide screens.
3. Use Visual > Cards > Layout > Auto-fit so cards resize with the page.

## 5. Finish
Add a page title text box, sync the Year and Hotel slicers (View > Sync slicers), save with Ctrl+S, and take screenshots for the project report.
