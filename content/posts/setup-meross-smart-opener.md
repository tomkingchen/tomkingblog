---
title: "Setup Meross Smart Garage Door Opener with Merlin Roller door"
date: 2024-12-27T20:56:55+11:00
draft: false
---

In this post, I’ll guide you through the steps I followed to set up the Meross Smart Garage Door Opener (MSG100) with the Merlin Roller Door (MR555MYQ). I hope it proves helpful to anyone with the same roller door model.

### What I got

- **Meross Smart Garage Door Opener (MSG100) with Homekit support**
- **Merlin MR555MYQ Roller Door Opener**
- **iPhone**

## Setting Up the Smart Garage Door Opener Using the Meross App

Once the Meross Smart Garage Door Opener is powered on, its LED will flash alternately between amber and green. Simply the prompts from the App to connect to the device's Wi-Fi network.

I did bump into an issue where the **Home** app stuck at "Loading Accessories and Scenes" screen. Below are the steps I tried to fix this.

1. Reconnect your iPhone to your regular Wi-Fi network.
2. Open the **Home** app.
3. If prompted with the loading screen, click **Reset Configuration**. This will reset the app.
    *(Note: If you haven’t set up anything with HomeKit before, this reset won’t cause any issues.)*

After resetting the HomeKit app, restart the setup process:

## Wiring the Smart Opener to the Roller Door

### Important Safety Reminder

Before you begin, **unplug both the smart opener and the roller door opener from power**.

### Step 1: Locate the Terminal Block

The terminal block on the Merlin MR555MYQ is covered by a rubber grommet. Remove it carefully using your fingers.

![image.png](https://blogfilesr2.tomking.xyz/rollerdoor_terminal.png)

### Step 2: Test the Ports

Once the grommet is removed, you’ll see four ports. The two rightmost ports are for door control.

1. Test the ports by shorting them with the small wire included in the Meross package.
2. This should trigger the door to open or close.

### Step 3: Connect the Wires

1. Punch holes in the rubber grommet to thread the two wires from the smart opener through it.
2. Match the cable colors to the terminal ports: **brown to brown**, **white to blue**.
3. Use a small flathead screwdriver to push the orange button under each port, insert the wire, and release the button to secure the wire.

![image.png](https://blogfilesr2.tomking.xyz/terminal_wiring.png)

Once the wiring is complete, push the rubber grommet back into place.

---

## Installing the Sensor

The Meross smart opener comes with a pair of magnetic sensors. These sensors are essential for detecting whether the door is open or closed.

### Step 1: Connect the Sensor

Attach the sensor cable to the smart opener as per the Meross instructions.

### Step 2: Position the Sensor

Positioning the sensor will depend on your specific garage and roller door setup.

💡 **Tip**: Test the roller door with the sensors in place a few times before securing them permanently. Misplaced sensors can cause the door to jam.

Here’s how I positioned mine:

![image.png](https://blogfilesr2.tomking.xyz/rollerdoor_sensors.png)

---

## Open Sesame 🎉

That’s it! Your Meross Smart Garage Door Opener should now be fully functional. You can open and close your garage door using the Meross app. Since my opener supports Homekit, I added it to the Home app and with that I can now control the garage door with my voice through Siri 🎉
