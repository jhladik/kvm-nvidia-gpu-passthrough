# KVM NVIDIA GPU Passthrough

Linux Host Setup Steps:

1. TBD

Set up Moonlight Logitech Extreme 3D joystick passhtrough on Moonlight:

1. Find where your `gamecontrollerdb.txt` is stored. On Mac OS, it is in `~/Library/Caches/Moonlight Game Streaming Project/Moonlight/gamecontrollerdb.txt`. The following exerpt will map the joystick axes and buttons to a virtual Xbox Controller which Moonlight can forward. You can then assign the virtual Xbox Controller in your flight simulator settings.
```
030000006d04000015c2000000000000,Logitech Extreme 3D,a:b2,b:b3,x:b4,y:b5,back:b0,guide:b1,start:b7,leftstick:b9,rightstick:b11,leftshoulder:b8,rightshoulder:b6,dpup:h0.1,dpdown:h0.4,dpleft:h0.8,dpright:h0.2,leftx:a0,lefty:a1,rightx:a2,righty:a3,platform:Windows,
030000006d04000015c2000000000000,Logitech Extreme 3D,a:b2,b:b3,x:b4,y:b5,back:b0,guide:b1,start:b7,leftstick:b9,rightstick:b11,leftshoulder:b8,rightshoulder:b6,dpup:h0.1,dpdown:h0.4,dpleft:h0.8,dpright:h0.2,leftx:a0,lefty:a1,rightx:a2,righty:a3,platform:Mac OS X,
030000006d04000015c2000000000000,Logitech Extreme 3D,a:b2,b:b3,x:b4,y:b5,back:b0,guide:b1,start:b7,leftstick:b9,rightstick:b11,leftshoulder:b8,rightshoulder:b6,dpup:h0.1,dpdown:h0.4,dpleft:h0.8,dpright:h0.2,leftx:a0,lefty:a1,rightx:a2,righty:a3,platform:Linux,
```
