# SmartCase

SmartCase is an interactive prototype of a smart carry-on suitcase designed to make traveling easier. The project focuses on common travel problems such as checking luggage weight, keeping track of packed items, locating a bag, and moving heavy luggage through an airport.

The interface includes both a display built into the suitcase and a connected phone interface.

## Features

- Built-in weight monitoring with overweight warnings
- Packing checklist for packed and missing items
- Lock and unlock controls
- Open/closed suitcase detection
- Bluetooth phone connection
- Find My suitcase tracking
- Distance-based Find My indicator
  - Green: suitcase is with you
  - Yellow: suitcase is nearby
  - Red: suitcase is separated
- Follow Mode for hands-free movement
- Travel Mode
- Battery monitoring
- USB-C charging
- Protective flap over the main display
- Multiple testing scenarios
- Airport Follow Mode simulation

## Testing UI

The right side of the application is used to simulate how SmartCase would behave if it were a real physical product.

The testing controls can be used to:

- Add or remove luggage weight
- Open and close the suitcase
- Add or remove packed items
- Lock and unlock the suitcase
- Connect or disconnect a phone
- Change the suitcase's distance from the user
- Test battery and USB-C charging
- Activate Travel Mode and Find My
- Load different travel scenarios
- Run the Follow Mode airport simulation

Changes made in the Testing UI are immediately reflected on the suitcase and phone interfaces.

## Technologies Used

- Svelte
- JavaScript
- HTML/CSS
- Vite
- Git and GitHub
- Vercel

## Running the Project Locally

Clone the repository:

```bash
git clone https://github.com/mariamalik2105/smart-suitcase.git
