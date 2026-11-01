![](../assets/Pasted%20image%2020261101141557.png)

Once the  scene has been modeled, world-coordinate positions are converted to viewing coordinates. The viewing-coordinate system is used in graphics packages as a reference for specifying the observer viewing position and the position of the projection plane.

Next, projection operations are performed to convert the viewing-coordinate description of the scene to coordinate positions on the projection plane, which will then be mapped to the output device. 

Objects outside the specified viewing limits are clipped h m further consideration, and the remaining objects are processed through visible-surface identification and surface-rendering procedures to produce the display within the device viewport.

### Properties of parallel projection

![](../assets/Pasted%20image%2020261101145219.png)
