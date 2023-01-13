


I was playing around with my Canon T6 camera and `gphoto2` for Python. 
The library worked great, but it was kind of finnicky to navegate the generated SWIG bindings and figure out what was there.
Here's a handy script that lists all the available configurations for a camera:

```python

import gphoto2 as gp

camera = gp.Camera()
camera.init()

def get_configs_dict(widget):
    """Builds a dictionary with the current and possible configurations for a given camera.

    Usage:
        >>> get_configs_dict(camera.get_config())
    """

    prop_data = {}
    for prop in "name", "type", "value", "choices", "label", "range":
        try:
            val = getattr(widget, f"get_{prop}")()
            if isinstance(val, gp.CameraWidgetChoiceIter):
                prop_data[prop] = [str(v) for v in val]
            else:
                prop_data[prop] = val
        except gp.GPhoto2Error as e:
            pass

    # Get children properties
    children = list(widget.get_children())
    if children:
        prop_data["children"] = [get_configs_dict(child) for child in children]

    return prop_data

from pprint import pprint
pprint(get_configs_dict(camera.get_config()))
```

And a JSON dump of what comes up for mine, as example output:

```json
{
  "name": "main",
  "type": 0,
  "label": "Camera and Driver Configuration",
  "children": [
    {
      "name": "actions",
      "type": 1,
      "label": "Camera Actions",
      "children": [
        {
          "name": "syncdatetimeutc",
          "type": 4,
          "value": 0,
          "label": "Synchronize camera date and time with PC (UTC)"
        },
        {
          "name": "syncdatetime",
          "type": 4,
          "value": 0,
          "label": "Synchronize camera date and time with PC"
        },
        {
          "name": "uilock",
          "type": 4,
          "value": 2,
          "label": "UI Lock"
        },
        {
          "name": "popupflash",
          "type": 4,
          "value": 2,
          "label": "Popup Flash"
        },
        {
          "name": "autofocusdrive",
          "type": 4,
          "value": 0,
          "label": "Drive Canon DSLR Autofocus"
        },
        {
          "name": "manualfocusdrive",
          "type": 5,
          "value": "None",
          "choices": [
            "Near 1",
            "Near 2",
            "Near 3",
            "None",
            "Far 1",
            "Far 2",
            "Far 3"
          ],
          "label": "Drive Canon DSLR Manual focus"
        },
        {
          "name": "cancelautofocus",
          "type": 4,
          "value": 0,
          "label": "Cancel Canon DSLR Autofocus"
        },
        {
          "name": "eoszoom",
          "type": 2,
          "value": "0",
          "label": "Canon EOS Zoom"
        },
        {
          "name": "eoszoomposition",
          "type": 2,
          "value": "0,0",
          "label": "Canon EOS Zoom Position"
        },
        {
          "name": "viewfinder",
          "type": 4,
          "value": 0,
          "label": "Canon EOS Viewfinder"
        },
        {
          "name": "eosremoterelease",
          "type": 5,
          "value": "None",
          "choices": [
            "None",
            "Press Half",
            "Press Full",
            "Release Half",
            "Release Full",
            "Immediate",
            "Press 1",
            "Press 2",
            "Press 3",
            "Release 1",
            "Release 2",
            "Release 3"
          ],
          "label": "Canon EOS Remote Release"
        },
        {
          "name": "eosmoviemode",
          "type": 4,
          "value": 2,
          "label": "Movie Mode"
        },
        {
          "name": "opcode",
          "type": 2,
          "value": "0x1001,0xparam1,0xparam2",
          "label": "PTP Opcode"
        }
      ]
    },
    {
      "name": "settings",
      "type": 1,
      "label": "Camera Settings",
      "children": [
        {
          "name": "datetimeutc",
          "type": 8,
          "value": 1673642635,
          "label": "Camera Date and Time"
        },
        {
          "name": "datetime",
          "type": 8,
          "value": 1673642635,
          "label": "Camera Date and Time"
        },
        {
          "name": "reviewtime",
          "type": 5,
          "value": "None",
          "choices": [
            "None",
            "2 seconds",
            "4 seconds",
            "8 seconds",
            "Hold"
          ],
          "label": "Quick Review Time"
        },
        {
          "name": "output",
          "type": 5,
          "value": "Off",
          "choices": [
            "TFT",
            "PC",
            "TFT + PC",
            "MOBILE",
            "TFT + MOBILE",
            "PC + MOBILE",
            "TFT + PC + MOBILE",
            "MOBILE2",
            "TFT + MOBILE2",
            "PC + MOBILE2",
            "TFT + PC + MOBILE2",
            "Off"
          ],
          "label": "Camera Output"
        },
        {
          "name": "movierecordtarget",
          "type": 5,
          "value": "None",
          "choices": [
            "None"
          ],
          "label": "Recording Destination"
        },
        {
          "name": "evfmode",
          "type": 5,
          "value": "1",
          "choices": [
            "1",
            "0"
          ],
          "label": "EVF Mode"
        },
        {
          "name": "ownername",
          "type": 2,
          "value": "",
          "label": "Owner Name"
        },
        {
          "name": "artist",
          "type": 2,
          "value": "Pedro von Hertwig",
          "label": "Artist"
        },
        {
          "name": "copyright",
          "type": 2,
          "value": "",
          "label": "Copyright"
        },
        {
          "name": "customfuncex",
          "type": 2,
          "value": "bc,4,1,2c,3,101,1,0,103,1,1,10f,1,0,2,2c,3,201,1,0,202,1,3,203,1,0,3,14,1,50e,1,3,4,38,4,701,1,1,704,1,0,70e,1,1,811,1,0,",
          "label": "Custom Functions Ex"
        },
        {
          "name": "focusinfo",
          "type": 2,
          "value": "eosversion=0,size=5184x3456,size2=5184x3456,points={{0,743,117,181},{-839,393,172,129},{839,393,172,129},{-1394,0,172,129},{0,0,224,222},{1394,0,172,129},{-839,-393,172,129},{839,-393,172,129},{0,-743,117,181}},select={},unknown={1000000 0ffff}",
          "label": "Focus Info"
        },
        {
          "name": "strobofiring",
          "type": 5,
          "value": "0",
          "choices": [
            "0",
            "1",
            "2"
          ],
          "label": "Strobo Firing"
        },
        {
          "name": "flashcharged",
          "type": 2,
          "value": "0",
          "label": "Flash Charging State"
        },
        {
          "name": "autopoweroff",
          "type": 2,
          "value": "240",
          "label": "Auto Power Off"
        },
        {
          "name": "depthoffield",
          "type": 2,
          "value": "0",
          "label": "Depth of Field"
        },
        {
          "name": "capturetarget",
          "type": 5,
          "value": "Internal RAM",
          "choices": [
            "Internal RAM",
            "Memory card"
          ],
          "label": "Capture Target"
        },
        {
          "name": "capture",
          "type": 4,
          "value": 0,
          "label": "Capture"
        },
        {
          "name": "remotemode",
          "type": 2,
          "value": "1",
          "label": "Remote Mode"
        },
        {
          "name": "eventmode",
          "type": 2,
          "value": "0",
          "label": "Event Mode"
        }
      ]
    },
    {
      "name": "status",
      "type": 1,
      "label": "Camera Status Information",
      "children": [
        {
          "name": "serialnumber",
          "type": 2,
          "value": "286755fad04869ca523320acce0dc6a4",
          "label": "Serial Number"
        },
        {
          "name": "manufacturer",
          "type": 2,
          "value": "Canon Inc.",
          "label": "Camera Manufacturer"
        },
        {
          "name": "cameramodel",
          "type": 2,
          "value": "Canon EOS Rebel T6",
          "label": "Camera Model"
        },
        {
          "name": "deviceversion",
          "type": 2,
          "value": "3-1.1.0",
          "label": "Device Version"
        },
        {
          "name": "vendorextension",
          "type": 2,
          "value": "",
          "label": "Vendor Extension"
        },
        {
          "name": "model",
          "type": 2,
          "value": "2147484676",
          "label": "Camera Model"
        },
        {
          "name": "ptpversion",
          "type": 2,
          "value": "256",
          "label": "PTP Version"
        },
        {
          "name": "",
          "type": 2,
          "value": "100%",
          "label": "Battery Level"
        },
        {
          "name": "batterylevel",
          "type": 2,
          "value": "100%",
          "label": "Battery Level"
        },
        {
          "name": "lensname",
          "type": 2,
          "value": "EF-S55-250mm f/4-5.6 IS II",
          "label": "Lens Name"
        },
        {
          "name": "eosserialnumber",
          "type": 2,
          "value": "133333333337",
          "label": "Serial Number"
        },
        {
          "name": "shuttercounter",
          "type": 2,
          "value": "46950",
          "label": "Shutter Counter"
        },
        {
          "name": "availableshots",
          "type": 2,
          "value": "35449",
          "label": "Available Shots"
        },
        {
          "name": "eosmovieswitch",
          "type": 2,
          "value": "0",
          "label": "Movie Switch"
        }
      ]
    },
    {
      "name": "imgsettings",
      "type": 1,
      "label": "Image Settings",
      "children": [
        {
          "name": "imageformat",
          "type": 5,
          "value": "RAW",
          "choices": [
            "Large Fine JPEG",
            "Large Normal JPEG",
            "Medium Fine JPEG",
            "Medium Normal JPEG",
            "Small Fine JPEG",
            "Small Normal JPEG",
            "Smaller JPEG",
            "Tiny JPEG",
            "RAW + Large Fine JPEG",
            "RAW"
          ],
          "label": "Image Format"
        },
        {
          "name": "imageformatsd",
          "type": 5,
          "value": "RAW",
          "choices": [
            "Large Fine JPEG",
            "Large Normal JPEG",
            "Medium Fine JPEG",
            "Medium Normal JPEG",
            "Small Fine JPEG",
            "Small Normal JPEG",
            "Smaller JPEG",
            "Tiny JPEG",
            "RAW + Large Fine JPEG",
            "RAW"
          ],
          "label": "Image Format SD"
        },
        {
          "name": "iso",
          "type": 5,
          "value": "Auto",
          "choices": [
            "Auto",
            "100",
            "200",
            "400",
            "800",
            "1600",
            "3200",
            "6400",
            "12800"
          ],
          "label": "ISO Speed"
        },
        {
          "name": "whitebalance",
          "type": 5,
          "value": "Auto",
          "choices": [
            "Auto",
            "AWB White",
            "Daylight",
            "Shadow",
            "Cloudy",
            "Tungsten",
            "Fluorescent",
            "Flash",
            "Manual"
          ],
          "label": "WhiteBalance"
        },
        {
          "name": "whitebalanceadjusta",
          "type": 5,
          "value": "0",
          "choices": [
            "-9",
            "-8",
            "-7",
            "-6",
            "-5",
            "-4",
            "-3",
            "-2",
            "-1",
            "0",
            "1",
            "2",
            "3",
            "4",
            "5",
            "6",
            "7",
            "8",
            "9"
          ],
          "label": "WhiteBalance Adjust A"
        },
        {
          "name": "whitebalanceadjustb",
          "type": 5,
          "value": "0",
          "choices": [
            "-9",
            "-8",
            "-7",
            "-6",
            "-5",
            "-4",
            "-3",
            "-2",
            "-1",
            "0",
            "1",
            "2",
            "3",
            "4",
            "5",
            "6",
            "7",
            "8",
            "9"
          ],
          "label": "WhiteBalance Adjust B"
        },
        {
          "name": "whitebalancexa",
          "type": 5,
          "value": "0",
          "choices": [
            "0",
            "1",
            "2",
            "3"
          ],
          "label": "WhiteBalance X A"
        },
        {
          "name": "whitebalancexb",
          "type": 5,
          "value": "0",
          "choices": [
            "0",
            "1",
            "2",
            "3"
          ],
          "label": "WhiteBalance X B"
        },
        {
          "name": "colorspace",
          "type": 5,
          "value": "AdobeRGB",
          "choices": [
            "sRGB",
            "AdobeRGB"
          ],
          "label": "Color Space"
        }
      ]
    },
    {
      "name": "capturesettings",
      "type": 1,
      "label": "Capture Settings",
      "children": [
        {
          "name": "exposurecompensation",
          "type": 5,
          "value": "0",
          "choices": [
            "0"
          ],
          "label": "Exposure Compensation"
        },
        {
          "name": "focusmode",
          "type": 5,
          "value": "Manual",
          "choices": [
            "One Shot",
            "AI Focus",
            "AI Servo",
            "Manual"
          ],
          "label": "Focus Mode"
        },
        {
          "name": "aspectratio",
          "type": 5,
          "value": "3:2",
          "choices": [
            "3:2",
            "4:3",
            "16:9",
            "1:1"
          ],
          "label": "Aspect Ratio"
        },
        {
          "name": "storageid",
          "type": 2,
          "value": "00020001",
          "label": "Storage Device"
        },
        {
          "name": "autoexposuremode",
          "type": 5,
          "value": "Manual",
          "choices": [
            "P",
            "TV",
            "AV",
            "Manual",
            "Bulb",
            "A_DEP",
            "DEP",
            "Custom",
            "Lock",
            "Green",
            "Night Portrait",
            "Sports",
            "Portrait",
            "Landscape",
            "Closeup",
            "Flash Off",
            "C2",
            "C3",
            "Creative Auto",
            "Movie",
            "Auto",
            "Handheld Night Scene",
            "HDR Backlight Control",
            "SCN",
            "Food",
            "Grainy B/W",
            "Soft focus",
            "Toy camera effect",
            "Fish-eye effect",
            "Water painting effect",
            "Miniature effect",
            "HDR art standard",
            "HDR art vivid",
            "HDR art bold",
            "HDR art embossed",
            "Panning",
            "HDR",
            "Self Portrait",
            "Hybrid Auto",
            "Smooth skin",
            "Fv"
          ],
          "label": "Canon Auto Exposure Mode"
        },
        {
          "name": "autoexposuremodedial",
          "type": 5,
          "value": "Manual",
          "choices": [
            "P",
            "TV",
            "AV",
            "Manual",
            "Bulb",
            "A_DEP",
            "DEP",
            "Custom",
            "Lock",
            "Green",
            "Night Portrait",
            "Sports",
            "Portrait",
            "Landscape",
            "Closeup",
            "Flash Off",
            "C2",
            "C3",
            "Creative Auto",
            "Movie",
            "Auto",
            "Handheld Night Scene",
            "HDR Backlight Control",
            "SCN",
            "Food",
            "Grainy B/W",
            "Soft focus",
            "Toy camera effect",
            "Fish-eye effect",
            "Water painting effect",
            "Miniature effect",
            "HDR art standard",
            "HDR art vivid",
            "HDR art bold",
            "HDR art embossed",
            "Panning",
            "HDR",
            "Self Portrait",
            "Hybrid Auto",
            "Smooth skin",
            "Fv"
          ],
          "label": "Canon Auto Exposure Mode Dial"
        },
        {
          "name": "drivemode",
          "type": 5,
          "value": "Continuous",
          "choices": [
            "Single",
            "Continuous",
            "Timer 10 sec",
            "Timer 2 sec",
            "Continuous timer"
          ],
          "label": "Drive Mode"
        },
        {
          "name": "picturestyle",
          "type": 5,
          "value": "User defined 3",
          "choices": [
            "Auto",
            "Standard",
            "Portrait",
            "Landscape",
            "Neutral",
            "Faithful",
            "Monochrome",
            "User defined 1",
            "User defined 2",
            "User defined 3"
          ],
          "label": "Picture Style"
        },
        {
          "name": "aperture",
          "type": 5,
          "value": "5.6",
          "choices": [
            "5.6",
            "6.3",
            "7.1",
            "8",
            "9",
            "10",
            "11",
            "13",
            "14",
            "16",
            "18",
            "20",
            "22",
            "25",
            "29",
            "32"
          ],
          "label": "Aperture"
        },
        {
          "name": "shutterspeed",
          "type": 5,
          "value": "1/200",
          "choices": [
            "bulb",
            "30",
            "25",
            "20",
            "15",
            "13",
            "10.3",
            "8",
            "6.3",
            "5",
            "4",
            "3.2",
            "2.5",
            "2",
            "1.6",
            "1.3",
            "1",
            "0.8",
            "0.6",
            "0.5",
            "0.4",
            "0.3",
            "1/4",
            "1/5",
            "1/6",
            "1/8",
            "1/10",
            "1/13",
            "1/15",
            "1/20",
            "1/25",
            "1/30",
            "1/40",
            "1/50",
            "1/60",
            "1/80",
            "1/100",
            "1/125",
            "1/160",
            "1/200",
            "1/250",
            "1/320",
            "1/400",
            "1/500",
            "1/640",
            "1/800",
            "1/1000",
            "1/1250",
            "1/1600",
            "1/2000",
            "1/2500",
            "1/3200",
            "1/4000"
          ],
          "label": "Shutter Speed"
        },
        {
          "name": "meteringmode",
          "type": 5,
          "value": "Evaluative",
          "choices": [
            "Evaluative",
            "Partial",
            "Center-weighted average"
          ],
          "label": "Metering Mode"
        },
        {
          "name": "liveviewsize",
          "type": 5,
          "value": "val 0",
          "choices": [
            "Large",
            "Medium",
            "Small"
          ],
          "label": "Live View Size"
        },
        {
          "name": "bracketmode",
          "type": 5,
          "value": "Unknown value 0000",
          "choices": [
            "Unknown value 0000"
          ],
          "label": "Bracket Mode"
        },
        {
          "name": "aeb",
          "type": 5,
          "value": "off",
          "choices": [
            "off",
            "+/- 1/3",
            "+/- 2/3",
            "+/- 1",
            "+/- 1 1/3",
            "+/- 1 2/3",
            "+/- 2"
          ],
          "label": "Auto Exposure Bracketing"
        },
        {
          "name": "alomode",
          "type": 5,
          "value": "Standard (disabled in manual exposure)",
          "choices": [
            "Standard (disabled in manual exposure)",
            "x1",
            "x2",
            "x3"
          ],
          "label": "Auto Lighting Optimization"
        }
      ]
    },
    {
      "name": "other",
      "type": 1,
      "label": "Other PTP Device Properties",
      "children": [
        {
          "name": "d402",
          "type": 2,
          "value": "Canon EOS Rebel T6",
          "label": "PTP Property 0xd402"
        },
        {
          "name": "d407",
          "type": 2,
          "value": "1",
          "label": "PTP Property 0xd407"
        },
        {
          "name": "d406",
          "type": 2,
          "value": "Unknown Initiator",
          "label": "PTP Property 0xd406"
        },
        {
          "name": "d303",
          "type": 2,
          "value": "1",
          "label": "PTP Property 0xd303"
        },
        {
          "name": "5001",
          "type": 6,
          "value": "100",
          "choices": [
            "100",
            "0",
            "75",
            "0",
            "50"
          ],
          "label": "Battery Level"
        }
      ]
    }
  ]
}
```

(NB: I've replaced serial number information above as it's not nice to have that out on the internet)
