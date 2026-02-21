## Project Overview
A compact ESP32 devboard designed for rapid prototyping of IoT devices.
The board functions similarly to an ESP32 DevKit, but it's smaller, more robust, and production-ready.

## Architecture 
The board is composed of the following functional blocks : 
- Power subsystem : TPS56339 Buck regulator
- Processing core : ESP32-WROOM-32E
- Communication Interfaces : USB, UART
- Protection : ESD, TVS, reverse polarity,
- Supporting circuitry : reset, reboot, level shifting,

## Key Engineering Decisions

* **Buck regulator instead of LDO**
  A switching buck regulator was selected to efficiently step down USB 5 V to 3.3 V while supporting the ESP32’s peak current demands. This avoids thermal limitations and voltage droop that would occur with a linear regulator.

* **Use of an ESP32 module rather than bare IC**
  The ESP32-WROOM module was chosen to simplify RF design, eliminate impedance-controlled antenna routing complexity, and reduce bring-up risk while maintaining full ESP32 functionality.

* **USB-C power input with CC resistors**
  A USB-C receptacle configured for default USB power mode allows reliable 5 V sourcing without requiring USB-PD negotiation, simplifying implementation while maintaining compatibility with modern cables and chargers.

* **Dedicated USB-UART bridge for development and debugging**
  A hardware USB-UART interface was included to provide stable flashing, logging, and recovery access independent of firmware state, improving development reliability.

* **Local decoupling strategy on all power domains**
  Multiple ceramic capacitors of varying values were placed near power pins to handle both high-frequency switching noise and transient load current from the ESP32 and digital interfaces.

* **Compact switching loop placement in the power stage**
  The regulator input capacitor, switching node, diode/MOSFET path, and inductor were positioned to minimize loop area and reduce switching noise and EMI risk.

* **2-layer PCB with continuous ground plane**
  A 2-layer stack was intentionally used to balance manufacturing cost and learning value while maintaining acceptable signal integrity through the use of a largely uninterrupted ground plane.

* **Testpoint exposure for critical rails and signals**
  Key nets such as VIN, 3V3, EN, IO0, and UART signals were exposed to simplify probing, debugging, and bring-up validation.

* **ESD and transient protection on USB data and power lines**
  TVS protection devices were added to improve robustness against cable hot-plugging and electrostatic discharge events.

* **Auto-programming transistor network for ESP32 boot control**
  A transistor-based control circuit was implemented to allow automatic toggling of EN and IO0 during flashing, enabling seamless firmware upload without manual button interaction.

* **Header-based GPIO breakout for prototyping flexibility**
  Dual 19-pin headers were used to expose ESP32 GPIOs, enabling breadboard compatibility and expansion without requiring board redesign.

* **Component package size compromise for assembly and density**
  0603 passives were selected as a balance between compactness and beginner-friendly solderability.


## Layout Strategy 
- functional placement performed before routing, prioritized distance optimization when it's critical.
- Power paths routed first with minimized loop area
- continuous ground plane on the second layer, while minimizing the usage of vias.
- sensitive and noisy circuits separated.
- Antenna keepout respected.

## Challenges & Mistakes Encountered

* **Overestimating component size during initial selection**
  The first chosen power inductor was significantly oversized for the board constraints, forcing a late replacement and footprint update. This highlighted the importance of checking mechanical dimensions early during component selection.

* **Uncertainty around decoupling placement and power node topology**
  Early schematic iterations created intermediate 3.3 V nodes before decoupling capacitors, raising concerns about transient stability for sensitive ESP32 pins. This led to a deeper understanding that decoupling effectiveness depends on physical placement rather than schematic net naming.

* **Confusion between hierarchical labels and true net organization**
  Heavy use of hierarchical labels initially gave a false sense of structured connectivity while increasing mental overhead. The design process clarified when hierarchical labels are necessary versus when global or local labels improve readability.

* **Footprint discovery and verification difficulties**
  Several selected components lacked easily accessible or clearly documented footprints, requiring manual footprint creation and verification. This emphasized the importance of mechanical datasheet review and footprint validation before committing to layout.

* **Ground routing misconceptions in 2-layer design**
  Early routing attempts treated ground as a signal to be manually routed, resulting in unnecessary vias and complexity. The transition to a ground-plane-first mindset significantly simplified routing and improved layout quality.

* **Switching regulator layout anxiety and via usage uncertainty**
  The switching regulator region introduced uncertainty regarding acceptable via placement and switching loop integrity. Iteration led to prioritizing loop compactness over aesthetic routing.

* **Difficulty managing dense silkscreen and annotation clutter**
  Default silkscreen settings produced excessive overlap and visual noise, leading to many DRC warnings. This required learning that silkscreen is secondary to electrical correctness and can be selectively simplified for readability.

* **Routing congestion between ESP32 and USB-UART interface**
  TX/RX and control signals initially became difficult to route due to placement decisions. This reinforced the importance of functional block placement before routing begins.

* **Misinterpretation of multi-pin power connectivity warnings**
  DRC warnings regarding internally connected power pins (e.g., multiple 3.3 V or VBUS pins) caused confusion, revealing the need to explicitly short these pins externally for proper current distribution and rule compliance.

* **Balancing perfectionism with forward progress**
  A recurring challenge was spending excessive time refining non-critical details (silkscreen, labeling, cosmetic routing). Progress improved once the focus shifted toward electrical correctness and manufacturability first.



  ## Manufacturing Readiness

This project was prepared with manufacturability and assembly considerations in mind, despite being developed primarily as a portfolio and learning exercise.

### PCB Fabrication Readiness

* **DRC clean:** The PCB passes electrical design rule checks with no critical errors. Remaining warnings are limited to silkscreen overlap and do not affect fabrication.
* **Standard 2-layer stackup:** The board was designed using a conventional 2-layer stack (signal + ground plane) to ensure compatibility with low-cost fabrication services.
* **Controlled trace widths:** Power traces (VBUS and 3.3 V) were routed with increased width to safely handle peak current and reduce voltage drop.
* **Ground plane implementation:** A continuous ground pour was used to improve return paths, reduce noise, and simplify manufacturing.

### Component Selection & Assembly

* **Widely available packages:** Components were chosen in common SMD footprints (0603, 0805, SOT-23, QFN-24, ESP32 module) to simplify sourcing and assembly.
* **Module-based RF design:** Using the ESP32-WROOM-32E avoids RF layout complexity and improves assembly reliability.
* **Hand-soldering feasibility:** Passive sizes and component spacing allow manual assembly, making the design accessible for prototyping and small-batch builds.
* **Polarized and orientation-sensitive parts labeled:** Diodes, regulators, and IC orientation were verified to reduce assembly risk.

### Power Integrity & Protection

* **Input protection included:** Polyfuse and TVS protection were added on VBUS to improve robustness against shorts and voltage transients.
* **Decoupling strategy:** Local decoupling capacitors were placed near regulators, microcontroller supply pins, and USB-UART IC to ensure stable operation.
* **Switching regulator layout considerations:** The switching loop was kept compact to reduce EMI and switching noise.

### Connector & Mechanical Considerations

* **Standard 2.54 mm pin headers:** Dual 19-pin headers were selected for compatibility with breadboards, jumper wires, and prototyping environments.
* **Accessible test points:** Key rails and signals (VIN, 3V3, EN, TX, RX, SW) include test pads to simplify debugging and validation.
* **USB-C mechanical footprint:** The USB connector footprint includes mounting pads for mechanical stability during insertion cycles.

### Generated Manufacturing Outputs

The following files can be exported directly for fabrication and assembly:

* Gerber files
* Drill files
* Pick-and-place (position) files
* Bill of Materials (BOM)
* Interactive 3D PCB view for mechanical validation

### Known Limitations

* Silkscreen density may require minor cleanup before production.
* No panelization strategy was defined.
* Assembly stencil optimization was not validated.
* RF performance and EMI compliance were not experimentally verified.

### Recommended Pre-Production Improvements

* Run a second DFM review focused on solder mask clearance and paste apertures.
* Validate USB differential impedance if targeting high-speed reliability.
* Perform prototype bring-up with thermal observation of the regulator and inductor.
* Add fiducials if automated assembly is planned.

