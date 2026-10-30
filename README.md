# AgriLink SL

**PROG101 Principles of Programming Logic and Design - Assignment 1**
Limkokwing University of Creative Technology, Sierra Leone Campus

- Student: [ARMANI C BECKLEY]
- Student ID: [905006175]
- Examiner: Elijah Fullah

## Overview

AgriLink SL is a logical design (no source code) for an open source SMS/USSD decision-support system that helps smallholder farmers in Sierra Leone decide **when to plant**, **where to sell** and **whether to store or sell** their crops. It works on any basic mobile phone, with no internet connection.

## Community problem

Smallholder farmers often lack timely information on planting windows and market prices. They plant late, sell at the first farm-gate offer and lose crops after harvest, which lowers their income and food security.

## Modules

| Module | Purpose |
|---|---|
| Main menu controller | Repeats a menu until the user exits |
| 1. registerFarmer() | Validates phone number and saves farmer details |
| 2. getCropAdvice() | Compares the current month with the crop's planting and harvest windows |
| 3. showBestMarket() | Finds the market with the best net price (price minus transport) |
| 4. sellOrStore() | Recommends SELL NOW or STORE using Boolean rules |

## Computational thinking and logic used

Decomposition, pattern recognition, abstraction and algorithm design; constants, variables, data types, Boolean logic, decision structures, repetition structures, functions and modular design.

## SDGs supported

SDG 2 Zero Hunger (primary), SDG 1 No Poverty, SDG 9 Industry, Innovation and Infrastructure, SDG 12 Responsible Consumption and Production.

## Open source and Digital Public Good

Proposed licences: Apache-2.0 (software) and CC BY 4.0 (data tables). The design aims to meet the Digital Public Goods Standard: open licence, SDG relevance, documentation, data portability, privacy protection and do-no-harm.

## Repository contents

- `AgriLinkSL_PROG101_Report.docx` - full report
- `pseudocode.md` - pseudocode for all modules
- `flowcharts/` - PNG images and Mermaid (`.mmd`) sources; GitHub renders Mermaid inside Markdown code blocks

## Data note

Crop windows, prices and distances in the report are illustrative test values and must be replaced with verified data from MAFFS, SLARI and market surveys before real use.
