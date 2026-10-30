# Pseudocode - AgriLink SL

All modules of the AgriLink SL logical design (PROG101 Assignment 1).

## Main algorithm

```text
ALGORITHM AgriLinkSL_Main
  // ----- Constants (values that never change) -----
  CONSTANT PHONE_LENGTH ← 8
  CONSTANT MAX_ATTEMPTS ← 3
  CONSTANT TRANSPORT_RATE ← 0.40      // Leones per bag per km
  CONSTANT PRICE_MARGIN ← 0.10        // minimum 10% gain to justify storing
  CONSTANT BULK_THRESHOLD ← 20        // bags
  CONSTANT MENU_EXIT ← 5
  // ----- Variables -----
  DECLARE choice AS Integer
  DECLARE running AS Boolean ← TRUE
  DECLARE lastBestMarket AS String ← ""     // shared result of getBestNetPrice()
  CALL loadData()
  WHILE running = TRUE DO
    DISPLAY "1 Register  2 Crop Advice  3 Best Market  4 Sell or Store  5 Exit"
    INPUT choice
    IF choice < 1 OR choice > MENU_EXIT THEN
      DISPLAY "Invalid choice, try again"
    ELSE IF choice = 1 THEN
      CALL registerFarmer()
    ELSE IF choice = 2 THEN
      CALL getCropAdvice()
    ELSE IF choice = 3 THEN
      CALL showBestMarket()
    ELSE IF choice = 4 THEN
      CALL sellOrStore()
    ELSE
      running ← FALSE
    END IF
  END WHILE
  DISPLAY "Thank you for using AgriLink SL"
END ALGORITHM
```

## Data loading

```text
FUNCTION loadData()
  // Data tables are stored in open files so extension officers can update them
  DECLARE CROP_NAME AS Array OF String          // Rice, Cassava, Groundnut, Maize
  DECLARE PLANT_START, PLANT_END AS Array OF Integer     // month numbers 1-12
  DECLARE HARVEST_START, HARVEST_END AS Array OF Integer
  DECLARE PERISHABLE AS Array OF Boolean
  DECLARE EXPECTED_PRICE AS Array OF Real       // expected price next month (Le per bag)
  DECLARE MARKET_NAME AS Array OF String        // Bo, Kenema, Freetown ...
  DECLARE PRICE AS 2D Array OF Real             // PRICE[crop][market], 0 = no data
  DECLARE DISTRICT_LIST AS Array OF String      // the 16 districts of Sierra Leone
  READ all tables from the data files
  SET totalCrops, totalMarkets
END FUNCTION
```

## Module 1 - Registration

```text
MODULE registerFarmer()
  DECLARE phoneNumber, farmerName, district AS String
  DECLARE mainCrop, attempts, i AS Integer
  DECLARE phoneOK AS Boolean ← FALSE
  attempts ← 0
  WHILE phoneOK = FALSE AND attempts < MAX_ATTEMPTS DO
    INPUT phoneNumber
    IF isValidPhone(phoneNumber) THEN
      phoneOK ← TRUE
    ELSE
      attempts ← attempts + 1
      DISPLAY "Invalid phone number"
    END IF
  END WHILE
  IF phoneOK = FALSE THEN
    DISPLAY "Too many invalid attempts. Try later."
    RETURN
  END IF
  IF farmerExists(phoneNumber) THEN
    DISPLAY "Welcome back, " + getFarmerName(phoneNumber)
    RETURN
  END IF
  INPUT farmerName
  REPEAT
    INPUT district
    IF NOT inList(district, DISTRICT_LIST) THEN
      DISPLAY "Unknown district"
    END IF
  UNTIL inList(district, DISTRICT_LIST)
  FOR i ← 1 TO totalCrops DO
    DISPLAY i + ". " + CROP_NAME[i]
  END FOR
  INPUT mainCrop
  saveFarmer(phoneNumber, farmerName, district, mainCrop)
  DISPLAY "Registration successful"
END MODULE

FUNCTION isValidPhone(phone) RETURNS Boolean
  IF LENGTH(phone) = PHONE_LENGTH AND isAllDigits(phone) THEN
    RETURN TRUE
  ELSE
    RETURN FALSE
  END IF
END FUNCTION
```

## Module 2 - Crop advice

```text
MODULE getCropAdvice()
  DECLARE cropName AS String
  DECLARE i, index, currentMonth AS Integer
  DECLARE found AS Boolean ← FALSE
  INPUT cropName
  i ← 1
  WHILE i ≤ totalCrops AND found = FALSE DO
    IF CROP_NAME[i] = cropName THEN
      found ← TRUE
      index ← i
    ELSE
      i ← i + 1
    END IF
  END WHILE
  IF found = FALSE THEN
    DISPLAY "Crop not supported"
    RETURN
  END IF
  currentMonth ← getCurrentMonth()
  IF currentMonth ≥ PLANT_START[index] AND currentMonth ≤ PLANT_END[index] THEN
    DISPLAY "PLANT NOW: " + cropName
  ELSE IF currentMonth < PLANT_START[index] THEN
    DISPLAY "Prepare land. Plant in " + (PLANT_START[index] − currentMonth) + " month(s)"
  ELSE
    DISPLAY "Planting window closed. Focus on weeding and crop care"
  END IF
  IF currentMonth ≥ HARVEST_START[index] AND currentMonth ≤ HARVEST_END[index] THEN
    DISPLAY "Harvest period: dry the crop well before storage"
  END IF
END MODULE
```

## Module 3 and core function - Best market

```text
FUNCTION getBestNetPrice(cropName, district) RETURNS Real
  DECLARE i AS Integer
  DECLARE price, distance, netPrice, bestNet AS Real
  bestNet ← 0
  lastBestMarket ← ""
  FOR i ← 1 TO totalMarkets DO
    price ← PRICE[cropIndex(cropName)][i]
    IF price > 0 THEN
      distance ← getDistance(district, MARKET_NAME[i])
      netPrice ← price − (distance × TRANSPORT_RATE)
      IF netPrice > bestNet THEN
        bestNet ← netPrice
        lastBestMarket ← MARKET_NAME[i]
      END IF
    END IF
  END FOR
  RETURN bestNet
END FUNCTION

MODULE showBestMarket()
  DECLARE cropName, district AS String
  DECLARE bestNet AS Real
  INPUT cropName, district
  bestNet ← getBestNetPrice(cropName, district)
  IF lastBestMarket = "" THEN
    DISPLAY "No price data available"
  ELSE
    DISPLAY "Best market: " + lastBestMarket
    DISPLAY "Net price: Le " + bestNet + " per bag"
  END IF
END MODULE
```

## Module 4 - Sell or store

```text
MODULE sellOrStore()
  DECLARE cropName, district, advice AS String
  DECLARE quantityBags AS Integer
  DECLARE hasStorage AS Boolean
  DECLARE currentNet, expectedPrice AS Real
  INPUT cropName, district, quantityBags, hasStorage
  currentNet ← getBestNetPrice(cropName, district)
  expectedPrice ← getExpectedPrice(cropName)
  IF isPerishable(cropName) THEN
    advice ← "SELL NOW (crop spoils quickly)"
  ELSE IF hasStorage = TRUE AND expectedPrice ≥ currentNet × (1 + PRICE_MARGIN) THEN
    advice ← "STORE and sell next month"
  ELSE
    advice ← "SELL NOW at best market"
  END IF
  IF quantityBags ≥ BULK_THRESHOLD THEN
    advice ← advice + ". Tip: join a cooperative to sell in bulk"
  END IF
  DISPLAY advice, lastBestMarket, currentNet, expectedPrice
END MODULE
```

## Helper functions

```text
FUNCTION getDistance(district, market) RETURNS Real      // looks up km in the distance table
FUNCTION getCurrentMonth() RETURNS Integer                // returns 1 to 12 from the system date
FUNCTION getExpectedPrice(cropName) RETURNS Real          // returns EXPECTED_PRICE[cropIndex(cropName)]
FUNCTION isPerishable(cropName) RETURNS Boolean           // returns PERISHABLE[cropIndex(cropName)]
FUNCTION cropIndex(cropName) RETURNS Integer              // linear search of CROP_NAME
FUNCTION inList(item, list) RETURNS Boolean               // linear search of a list
FUNCTION farmerExists(phone) RETURNS Boolean              // checks the farmer file
```
