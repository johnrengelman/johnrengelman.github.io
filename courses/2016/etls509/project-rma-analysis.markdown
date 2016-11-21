---
layout: page
title: ETLS 509 - V&V (Fall 2016) Project RMA Analysis
---

# Summary

A critical aspect of the MPALSS system is to provide life supporting atmosphere
for a stranded astronaut for the extended mission time frame. Life supporting
atmosphere consists of 2 primary components: supply of oxygen and removal of
carbon dioxide.

The Mars Habitat provides atmosphere through the Oxygenator. The Oxygenator
functions by separating the gas atmosphere in a chemical reaction to multiple
components including carbon dioxide, water vapor, and other trace components.
The water vapor is then condensed to liquid water, filtered, and then separated
into hydrogen and oxygen through electrolysis. The gaseous oxygen is then 
crygenically cooled to liquid form before being stored in tanks for later use.

The Oxygenator system diagram is show below:
![Oxygenator System Diagram](files/Oxygenator.png)

The current Oxygenator system is designed with singular components for each
item in the diagram. Each of the Oxygenator components is manufactured to meet 
a specific failure rate. The overall system system is designed to meet the
standard Mars Surface Mission duration time with a 2 - 3 time buffer.
The MTBFs for each component is provided in the following table:

| Component          | MTBF          |
|:-------------------|:--------------|
| Atmospheric Sensor | 50,000 hours  |
| Pump               | 20,000 hours  |
| Valve              | 40,000 hours  |
| Hydrogen Tank      | 100,000 hours |
| Reactor Chamber    | 30,000 hours  |
| Condenser          | 30,000 hours  |
| Gas Membrane       | 40,000 hours  |
| Coolant Lines      | 100,000 hours |
| Coolant Radiator   | 100,000 hours |
| Water Filter       | 30,000 hours  |
| Water Electrolyzer | 30,000 hours  |
| Crygenic Cooler    | 30,000 hours  |
| Liquid Oxygen Tank | 100,000 hours |
{:.bordertable}

# Requirements

As part of the MPALSS system proposal, provide a full RMA analysis of the
Oxygenator system. The RMA analysis shall show the statistical analysis
to determine the MTBF for the current system design.
The analysis shall propose a redesigned Oxygenator system to achieve the MTBF
required for the MPALSS extended mission duration.
The redesigned system shall contain the same system components as the original
system.
