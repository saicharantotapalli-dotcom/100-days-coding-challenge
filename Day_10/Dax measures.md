##### **Calculated columns (DAX)**

##### 

##### **create in Data view when needed**



(Use calculated columns for row-level values used in table visuals and conditional formatting.)



-- DelayDays (if you prefer DAX calculated column instead of Power Query)

DelayDays = DATEDIFF( 'logistics\_data'\[ExpectedDate], 'logistics\_data'\[DeliveryDate], DAY )





-- IsCancelled flag

IsCancelled = IF( 'logistics\_data'\[DeliveryStatus] = "Cancelled", 1, 0 )





-- IsOnTime flag (treat <=0 delay as on-time)

IsOnTimeFlag = VAR \_status = TRIM(LOWER('logistics\_data'\[DeliveryStatus])) 

&nbsp;              VAR \_delay = 'logistics\_data'\[DelayDays] 

&nbsp;              RETURN IF(\_status = "cancelled",0,IF(\_status = "on-time" || \_delay <= 0,1,0))





##### **Core DAX measures**

##### 

##### **I've written concise measures and small helper measures.**



-- 1) Total Shipments (unique)

Total Shipments = DISTINCTCOUNT( 'logistics\_data'\[ShipmentID] )



-- 2) Total Delivery Cost

Total Delivery Cost = SUM( 'logistics\_data'\[Cost\_USD] )



-- 3) Avg Delivery Time (days) — overall (filter context aware)

Avg Delivery Time (days) = AVERAGE( 'logistics\_data'\[DeliveryTime\_Days] )



-- 4) On-time deliveries (count)

On-time Deliveries = 

CALCULATE(

&nbsp;   DISTINCTCOUNT( 'logistics\_data'\[ShipmentID] ),

&nbsp;   'logistics\_data'\[DeliveryStatus] = "On-time"

)



-- 5) On-time Delivery % (excludes cancelled automatically if you prefer)

On-time Delivery % = 

VAR OnTime = \[On-time Deliveries]

VAR Total = \[Total Shipments]

RETURN DIVIDE( OnTime, Total, 0 )



-- 6) Delayed shipments (count, > 0 delay)

Delayed Shipments = 

CALCULATE(

&nbsp;   DISTINCTCOUNT( 'logistics\_data'\[ShipmentID] ),

&nbsp;   'logistics\_data'\[DelayDays] > 0

)



-- 7) Shipments delayed > 2 days

Delayed > 2 days = 

CALCULATE(

&nbsp;   DISTINCTCOUNT( 'logistics\_data'\[ShipmentID] ),

&nbsp;   'logistics\_data'\[DelayDays] > 2

)



-- 8) Avg Delivery Time per Region (works when Region is in visual; preserves region filter)

Avg Delivery Time per Region = AVERAGE( 'logistics\_data'\[DeliveryTime\_Days] )



-- 9) Delayed % (overall)

Delayed % = DIVIDE( \[Delayed Shipments], \[Total Shipments], 0 )











