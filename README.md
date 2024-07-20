# Switching XY and Moving Origin with config changes rather than port switching.
All changes have the originall values near them. 
##  Changes in printer.cfg 
### 1. [quad_gantry_level]
- Change the `gantry_corners` to fit the coordinate system (choose either XY flipped values).
- Change the QGL `points` (flip and rotate)
```
[quad_gantry_level]
gantry_corners:
  # XY flipped lead screw positions
  -80.8,53.4
  430.0,393.5
  # normal lead screw positions
  #-93.5,-80.8
  #353.4,430.0
  # default XY flipped corners
  #-35.8,-112.8
  #367.8,354.8
  # Magneto x default corners
  #-54.8,-35.8
  #412.8,367.8
points:
  25,25
  25,290
  380,290
  380,25
  #25,25
  #25,380
  #290,380
  #290,25

speed: 250
horizontal_move_z: 5
retries: 3
retry_tolerance: 0.0325
max_adjust: 4
```
### 2. XY stepper changes (`[stepper_y]` an `[stepper_x]`)
- Change `[stepper_y]` to `[stepper_x]` and vice versa.
- For the new `[stepper_y]` (Driver 1): `position_endstop` should be your maximum Z value. (315 for me, yours might be different. 310 should be a safe value)
```
# Driver1
[stepper_y] #stepper_x # 300mm axis, stepper_x by default.  Currently referred to as the X axis by the LINEAR_MOTOR_CONTROL KlipperScreen and the MagnetoSuperTool. (x,y is defined by port on the ESP32 and can not be redefined AFAIK)
step_pin: PF13
dir_pin: PF12 #!PF12
enable_pin: !PF14
microsteps: 16
endstop_pin: ^!PE8
rotation_distance: 3.2
step_pulse_duration: 0.0000002
position_endstop: 315 #0
position_max: 315
homing_speed: 50

# Driver0
[stepper_x] #stepper_y # 400mm axis, stepper_y by default.  Currently referred to as the Y axis by the LINEAR_MOTOR_CONTROL KlipperScreen and the MagnetoSuperTool. (x,y is defined by connection to the ESP32 and can not be redefined AFAIK)
step_pin: PG0
dir_pin: !PG1
enable_pin: !PF15
step_pulse_duration: 0.0000002
microsteps: 16
endstop_pin: ^!PE9
rotation_distance: 3.2
position_endstop: 0
position_max: 400
homing_speed: 50
```
### 3. [homing_override] (Needed if you are using the load cell. Can be commented out if using Beacon.)
- Flip the X and Y values for the `G0` command.
```
[homing_override]
axes: xyz
set_position_z: 0
gcode:
  LM_ENABLE
  {% if not 'Z' in params and not 'Y' in params and 'X' in params %}
    G28 X
  {% elif not 'Z' in params and not 'X' in params and 'Y' in params %}
    G28 Y    
  {% elif not 'Z' in params and 'X' in params and 'Y' in params %}
    G28 X
    G28 Y
  
  {% else %}
    G90
    G1 Z5
    G28 X
    G28 Y
    G0 X200 Y150 F6000
    G4 P3000
    #LC28
    G28 Z
    G1 Z5
  {% endif %}
```
### 4. [bed_mesh]
- Flip the `mesh_max` coordinates.
- (Beacon only) Flip the `zero_reference_position` coordinates.
```
[bed_mesh]
mesh_min: 0, 0
mesh_max: 359,300 #300,359
speed: 150
horizontal_move_z: 2
probe_count: 25,25
algorithm: bicubic
split_delta_z: 0.0125
move_check_distance: 3
mesh_pps: 2,2
fade_start: 0
fade_end: 3
fade_target: 0
zero_reference_position: 200, 150 #150,200
```


### 5. Z stepper changes
- We want to rotate the z stepper definitions counter clockwise so that `stepper_z` is on the front left when you are facing the printer screen.
- Please take note for the homing_retract_dist on `[stepper_z]` to make sure you are using the right one for you sensor.
- `step_pin`, `dir_pin`, and `enable_pin` under `[stepper_z]` should be copied to the same variables in `[stepper_z1]`
- `step_pin`, `dir_pin`, and `enable_pin` under `[stepper_z1]` should be copied to the same variables in `[stepper_z2]`
- `step_pin`, `dir_pin`, and `enable_pin` under `[stepper_z2]` should be copied to the same variables in `[stepper_z3]`
- `step_pin`, `dir_pin`, and `enable_pin` under `[stepper_z3]` should be copied to the same variables in `[stepper_z]`
```
## Z0 Stepper - Front Left on MOTOR2_1
[stepper_z]
step_pin: PC13 #PF11
dir_pin: !PF0 #!PG3
enable_pin: !PF1 #!PG5
endstop_pin: probe:z_virtual_endstop
rotation_distance: 4
microsteps: 16

#endstop_pin: PG10
##  Z-position of nozzle (in mm) to z-endstop trigger point relative to print surface (Z0)
##  (+) value = endstop above Z0, (-) value = endstop below
##	Increasing position_endstop brings nozzle closer to the bed
##  After you run Z_ENDSTOP_CALIBRATE, position_endstop will be stored at the very end of your config
# position_endstop: -1.0
##--------------------------------------------------------------------

##	Uncomment below for 250mm build
#position_max: 240

##	Uncomment below for 300mm build
position_max: 300

##	Uncomment below for 350mm build
#position_max: 340

##--------------------------------------------------------------------
position_min: -5
homing_speed: 3
second_homing_speed: 2
# homing_retract_dist: 0 # use this for beacon
# homing_retract_dist: 3 # use this for default loadcell

##	Make sure to update below for your relevant driver (2208 or 2209)
[tmc2209 stepper_z]
uart_pin: PE4 #PC6
interpolate: true
run_current: 0.9
hold_current: 0.5
sense_resistor: 0.110
stealthchop_threshold: 999999

##	Z1 Stepper - Rear Left on MOTOR3
[stepper_z1]
step_pin: PF11 #PG4
dir_pin: !PG3 #!PC1
enable_pin: !PG5 #!PA2
rotation_distance: 4
microsteps: 16

##	Make sure to update below for your relevant driver (2208 or 2209)
[tmc2209 stepper_z1]
uart_pin: PC6 #PC7
interpolate: true
run_current: 0.9
hold_current: 0.5
sense_resistor: 0.110
stealthchop_threshold: 999999

##	Z2 Stepper - Rear Right on MOTOR4
[stepper_z2]
step_pin: PG4 #PF9
dir_pin: !PC1 #!PF10
enable_pin: !PA2 #!PG2
rotation_distance: 4
microsteps: 16

##	Make sure to update below for your relevant driver (2208 or 2209)
[tmc2209 stepper_z2]
uart_pin: PC7 #PF2
interpolate: true
run_current: 0.9
hold_current: 0.50
sense_resistor: 0.110
stealthchop_threshold: 999999

##	Z3 Stepper - Front Right on MOTOR5
[stepper_z3]
step_pin: PF9 #PC13
dir_pin: !PF10 #!PF0
enable_pin: !PG2 #!PF1
rotation_distance: 4
microsteps: 16

##	Make sure to update below for your relevant driver (2208 or 2209)
[tmc2209 stepper_z3]
uart_pin: PF2 #PE4
interpolate: true
run_current: 0.9
hold_current: 0.50
sense_resistor: 0.110
stealthchop_threshold: 999999
```

### 7. [input_shaper]
- Switch the values for `shaper_type_y` and `shaper_type_x`
- Switch the values for `shaper_freq_y` and `shaper_freq_x`
```
[input_shaper]
shaper_type_y: ei #zv
shaper_freq_y : 44.4 #27.0
shaper_type_x: zv #ei
shaper_freq_x: 27.0 #44.4
```

### If you have a beacon, flip the `home_xy_position` coordinates and switch the `x_offset` and `y_offset`.
## At this point make sure all your macros have the coordinates flipped. (Check your Macros.cfg, Beacon.c, etc)
## After saving and restarting, KlipperScreen and Mainsail X,Y move buttons should move in a more intutive direction. The origin should also be in the front left corner now.

## OrcaSlicer Changes
1. Go to the printer settings -> Printable area
   
![image](https://github.com/user-attachments/assets/e2a98610-7cca-4ff9-8a5b-b67313509358)
- Change size from `X 300` and `Y 400` to `X 400` and `Y 300`
- Download `magnetox_model_texture-400x300.png` and move the texture file to a safe location. Load the texture from that location.
- Download `magnetox_mode-400x300l.stl` and move the bed STL file to a safe location. Load the STL from that location. 

![image](https://github.com/user-attachments/assets/f2132dfb-ddd0-40b2-8ce4-0bbdc48b4ccd)

- Your Orcaslicer shoud now have the origin in the front left corner and the plate should be faced correctly.
![image](https://github.com/user-attachments/assets/cc4f8c48-c080-4c15-9e00-9f9b984db02c)



