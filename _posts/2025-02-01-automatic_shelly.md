---
layout: post
title: "Automatic Pool Pump"
date: 2025-05-13
categories: [automation]
tags: [home-assistant, python, zigbee]
author: Lucas
---

# Main Goal

This project started with a very normal problem in my family.

A pool pump has to run for a few hours every day so the water stays clean enough. Before this project, this was done with a normal plug timer. This worked, but it also meant that the pump always ran at the configured time, no matter if electricity was cheap or expensive at that moment.

Because the electricity price changes during the day, I wanted to automate this a bit. The goal was not to build a huge smart home system. The goal was to run the pool pump during cheaper hours and still keep it simple enough that a non technical person in my family could use it.

![Tools and parts during the pool pump automation setup](/assets/img/shelly_plug_assembly.jpg)
*Preparing the outdoor plug and wiring near the pool.*

# First Version

My first idea was to use my existing Home Assistant instance at home. I connected the two Fritzboxes with a VPN, so the smart plug in the other network could be seen and controlled from my Home Assistant.

This was a good first test, because I learned how remote control over VPN can work in Home Assistant. But it was also not the best long term setup.

# How The Automation Works

The automation is based on a small Python script. Every day it gets the electricity prices from the aWATTar API. It then selects the cheapest hours for the pool pump, based on the number of hours the pump should run.

The running hours are not hard coded. I added an input field in the Home Assistant dashboard, so the family member can change the needed runtime when it is necessary. For example, if the pool needs more filtering, the value can be increased before the next daily schedule is created.

The script also does one extra thing: it slightly favors daytime hours. The reason is simple. The pool pump makes noise, and it should not disturb the neighbours during the night if this can be avoided. Night hours are not completely excluded, because sometimes the electricity price can be low enough to still make sense. They just need to be cheaper to be selected.

After the best time slots are found, the script creates the schedule in Home Assistant. The family member can then see if the pump is running and also see the automatic schedule times for the day. If something is wrong, the script writes information to a log file.

# Problems With The First Setup

The first version worked, but it had too many weak points.

The VPN connection between the Fritzboxes over IPsec was not stable enough during the first test period. Sometimes this meant that the remote device was not reachable from my Home Assistant instance.

Another problem was the connection outside. The first setup depended on WiFi and an outdoor range extender. The distance to the plug was around 30 meters, and the connection was not reliable enough for something that should run every day without manual checks.

# Final Setup

The better solution was to move the automation closer to the pool.

I installed a Raspberry Pi 4B locally and used it to run Home Assistant at the same place as the pool pump. The pump is now controlled with a Zigbee plug, and an outdoor range extender helps to keep the Zigbee connection stable.

![Pool pump installation with filter system and outdoor box](/assets/img/final_pool_setup.jpg)
*The pool pump setup after moving the automation closer to the actual device.*

The Python script runs daily as a cron job on the Raspberry Pi. It gets the prices, checks the running hours from Home Assistant, creates the schedule, and then Home Assistant controls the plug during the day.

This setup removed the need to control the device over a VPN. It also made the whole system easier to reason about, because the important parts are now local.

# What I Learned

The biggest learning for me was that a first working idea is not always the best long term solution. The VPN version was interesting and helped me understand remote control with Home Assistant, but it added too many possible failure points.

The final setup is much simpler in daily use. A Raspberry Pi, Home Assistant, a Zigbee plug, a dashboard input, and a Python script are enough to solve a real problem. I also liked that the solution is useful for someone who does not care about the technical details. They can see the schedule, change the runtime if needed, and otherwise just let it run.

I do not have measured savings yet, so I do not want to claim a specific amount. But the pump now runs more intelligently than before, because it is scheduled around the daily electricity prices instead of a fixed timer.
