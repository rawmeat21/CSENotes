![[Pasted image 20260617214242.png]]

**Physical Address**

The physical address is the lowest level address and is also referred as link address. The physical address of a node is defined by its LAN. The physical address is included in the frame by the data link layer. Physical Address is 48 bits address. First 24 bits is decided by OUI and Lase 24 bits is decided by Vendor/Manufacturer of device.

![[Pasted image 20260617214357.png]]

At data link layer, the frame contains physical addresses in the header. The data link layer at sender receives data from upper layer, encapsulates the data in a frame, adds a header.

![[Pasted image 20260617214414.png]]



**Logical Address**

Logical addresses are independent of underlying physical networks.

![[Pasted image 20260617214452.png]]

It is a 32-bit address which uniquely defines host connected to Internet. The physical addresses change from hop to hop, but the logical address usually remains the same.

![[Pasted image 20260617214519.png]]

Since different networks can have different address formats hence a universal address system is required which can identify each host uniquely of underlying physical networks. 


**Post Address**

Port is communication end point. Use of port address (port number) is to process to process communication in network. Port address is 16 bit address.

![[Pasted image 20260617214629.png]]


**Specific Address**

Specific addresses are designed by users for some applications. 

For example, [www.facebook.com](file:///D:/GMIT_Data/Teaching%20Learning/Video%20making%20format/Video%20making%20pending/www.facebook.com). 

It is the example of Universal Resource Locator (URL). It is used is used to find a document on the world wide web.

![[Pasted image 20260617214721.png]]

Another example, [xyz@gmail.com](mailto:xyz@gmail.com). It is the example of e-mail address. Email is used send text and multimedia files over internet to particular user.

![[Pasted image 20260617214739.png]]

The specific addresses get changed to corresponding port and logical addresses by the station or host who sends it.


