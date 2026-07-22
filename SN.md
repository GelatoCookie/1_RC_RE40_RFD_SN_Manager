# Getting Serial Number from Zebra RFID API

There are two primary ways to retrieve serial numbers using the Zebra RFID API: during the **Discovery Phase** (from the available reader list) and during the **Connected Phase** (from the reader's radio capabilities).

## 1. Discovery Phase: `ReaderDevice.serialNumber`

When you are scanning for available readers using `GetAvailableRFIDReaderList()`, you can access the serial number before establishing a full connection. This is useful for identifying hardware (e.g., distinguishing between multiple RFD40 sleds).

```kotlin
val readers = Readers(context, ENUM_TRANSPORT.ALL)

try {
    val availableReaders: List<ReaderDevice> = readers.GetAvailableRFIDReaderList()
    
    availableReaders.forEach { device ->
        // The serial number is a property of the ReaderDevice object
        val deviceSN: String = device.serialNumber
        val deviceName: String = device.name
        
        Log.d("RFID", "Found Reader: $deviceName | SN: $deviceSN")
        
        // Example: check for a specific Zebra device
        if (deviceName.startsWith("RFD40") || deviceName.startsWith("TC")) {
            println("Target device found with SN: $deviceSN")
        }
    }
} catch (e: Exception) {
    Log.e("RFID", "Discovery failed", e)
}
```

---

## 2. Connected Phase: `ReaderCapabilities.serialNumber`

Once you have selected a `ReaderDevice` and successfully called `connect()`, you can retrieve the internal radio serial number. This is the authoritative serial number from the RFID module.

```kotlin
// Connection should be performed in a background thread
CoroutineScope(Dispatchers.IO).launch {
    try {
        val device: ReaderDevice = availableReaders[0] 
        val reader: RFIDReader = device.rfidReader
        
        if (!reader.isConnected) {
            reader.connect()
            
            // Accessing the serial number via ReaderCapabilities
            // Note: This requires the reader to be CONNECTED
            val radioSN = reader.ReaderCapabilities.serialNumber
            
            Log.d("RFID", "Connected Reader Radio SN: $radioSN")
            Log.d("RFID", "Host Name: ${reader.hostName}")
            Log.d("RFID", "Model Name: ${reader.ReaderCapabilities.modelName}")
        }
    } catch (e: InvalidResponseException) {
        Log.e("RFID", "Connection failed: Invalid response", e)
    } catch (e: OperationFailureException) {
        Log.e("RFID", "Connection failed: ${e.results}", e)
    }
}
```

---

## 3. Alternative: `RFIDReader.serialNumber` (Version Dependent)

In some versions of the Zebra RFID SDK, the `RFIDReader` object itself may expose a `serialNumber` property as a convenience wrapper for the capability. However, using `ReaderCapabilities` is the most reliable cross-version method.

```kotlin
// If available in your SDK version:
val sn = reader.serialNumber 
```

---

## Summary Comparison

| Access Point | Object | Property | State Required |
| :--- | :--- | :--- | :--- |
| **Discovery** | `ReaderDevice` | `.serialNumber` | No connection needed |
| **Connected** | `RFIDReader` | `.ReaderCapabilities.serialNumber` | Must be Connected |

## Best Practices
1. **List Selection**: Use `ReaderDevice.serialNumber` to show the user which device they are about to connect to.
2. **Identification**: Use `ReaderCapabilities.serialNumber` for logging or back-end registration after the connection is solid.
3. **Safety**: Always check `reader.isConnected` before accessing `ReaderCapabilities` to avoid `InvalidResponseException`.
