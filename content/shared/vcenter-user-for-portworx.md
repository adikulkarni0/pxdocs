You will need to provide Portworx with a vCenter server user that has the following minimum [vSphere privileges](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.security.doc/GUID-FEAB5DF5-F7A2-412D-BF3D-7420A355AE8F.html):

* Datastore
    * Allocate space
    * Browse datastore
    * Low level file operations
    * Remove file
* Host
    * Local operations
    * Reconfigure virtual machine
* Virtual machine
    * Change Configuration
    * Add existing disk
    * Add new disk
    * Add or remove device
    * Advanced configuration
    * Change Settings
    * Extend virtual disk
    * Modify device settings
    * Remove disk

If you create a custom role as above, make sure to select "Propagate to children" when assigning the user to the role. 