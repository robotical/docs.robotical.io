# MartyV2 Docs Site Structure

The skeleton for the Marty V2 Docs website is as follows:

```bash
📦martyv2
 ┣ 📂calibration
 ┃ ┣ 📜calibration-troubleshooting.html
 ┃ ┗ 📜index.html
 ┣ 📂communication
 ┃ ┣ 📂bluetooth
 ┃ ┃ ┣ 📜bluetooth.md
 ┃ ┃ ┗ 📜index.html
 ┃ ┗ 📂RIC-rest
 ┃ ┃ ┣ 📜index.html
 ┃ ┃ ┗ 📜RIC-rest.md
 ┣ 📂hardware
 ┃ ┣ 📂addon
 ┃ ┃ ┣ 📜addon.md
 ┃ ┃ ┗ 📜index.html
 ┃ ┣ 📂RIC
 ┃ ┃ ┣ 📜index.html
 ┃ ┃ ┗ 📜RIC.md
 ┃ ┣ 📂specs
 ┃ ┃ ┣ 📜index.html
 ┃ ┃ ┗ 📜specs.md
 ┃ ┣ 📜index.html
 ┃ ┗ 📜RIC.html
 ┣ 📂react-native
 ┃ ┣ 📜index.html
 ┃ ┗ 📜react-native.md
 ┣ 📂release-notes
 ┃ ┗ 📜index.html
 ┗ 📂scratch
 ┃ ┣ 📜index.html
 ┃ ┗ 📜scratch.md
```
**Note: The above tree structure can be created using the *file-tree-generator* extension for Visual Studio Code**

There is an initial index.html at the root repository which is the main landing page for the user for the Marty V2 material of the website. There is a blue button at the left of the main text titled 'Marty V1 Docs' which provides a link to the old Marty V1 website.

The layout of the website is similar to the older version for consistency, however, the previous subsections:
- Standard Setup with Just Rick
- RaspberryPi in head with Rick
- Simulation

are now replaced with:
- Bluetooth to RIC
- Marty App

These new sections provides links to documentations/guides on the Scratch interface for Bluetooth to RIC and information about the Marty App that's implemented using React Native respectively.
