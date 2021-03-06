# Game Save

An extendable save system for Unity.

## Installation
Follow the steps [Here](https://github.com/FedoraDevStudios/Installation-Unity) to add this package to your Unity project using this package's URL.

##### Json.NET for Unity
Next, you will need to add [Json.NET for Unity](https://github.com/jilleJr/Newtonsoft.Json-for-Unity). Add `https://github.com/jilleJr/Newtonsoft.Json-for-Unity` in the `Repository URL` box and hit `Find Versions`. Select the latest `upm` version, i.e. `10.0.302 - upm` and then `Install Package`.

## Usage
### Add to Scene
For the easiest use, add a `GameSaveBehaviour` component to a game object in the scene. Then, select `DictionaryGameSave` in the drop down on the component. Next, select the `JsonDataHandler` in the `Data Handler` drop down. This can be changed later if you create your own handler. Finally, you can start adding entries in the `Game Data` dropdown. At first, you won't have anything to add until you create your own `IGameData` instances.

### Demo
As an example, go ahead and create a new object and add the `GameDataBehavour` component to it. You can select the data type from the `Game Data` drop down. For now, just use the `ExampleData` option and add any dummy values you wish to add. Next, in the `GameSaveBehaviour`, add an entry with any text as the key, then drag the `GameDataBehaviour` component into the value slot and hit the `Add` button. Finally, add a `TestingBehaviour` from the Example provided and assign the `GameSaveBehaviour` component to the `Game Save` slot. Hit `Save` and you'll find the save file is created. You can show the file in your file browser by clicking `Show File Location` in the `GameSaveBehaviour` > `File Options` drop down. Finish testing the functionality by changing some values in the data component and then hitting `Load` to see the values return to the saved ones.

### Explanation
Game Save is responsible for taking an arbitrary amount of data and storing it to disk. It will manage how and where that data is stored. You will be responsible for how the data is layed out on a byte-to-byte basis. Luckily, I've also included a Byte Manager to handle all the conversion nonsense for you.

#### IGameSave
This interface brings all of the others together. It stores a list of `IGameData` that gets run through the `IDataHandler` and optionally through `IEncrypt` before storing the data to a file on disk. You can implement your own version of this interface, however the included one comes with a lot of options.

#### IGameData
This interface should be implemented to enable an object to be saved to disk. This interface only requires 2 methods to be defined; `SaveData` and `LoadData`. Below are the implementations used in the `ExampleData` object in the included example.

```C#
[Serializable]
public class ExampleData : IGameData
{
	// Set up a byte manager in the inspector
	[SerializeField] IByteManager _byteManager;

	// These members will be saved to disk
	[SerializeField] int _sampleInt;
	[SerializeField] float _sampleFloat;
	[SerializeField] string _sampleString;

	public void LoadData(byte[] data)
	{
		// Assign byte data to the byte manager
		_byteManager.SetByteArray(data);

		// Pull elements from the byte manager in the same order they get saved in
		_sampleInt = _byteManager.GetInt();
		_sampleFloat = _byteManager.GetFloat();
		_sampleString = _byteManager.GetString();
	}

	public byte[] SaveData()
	{
		// Initialize the byte manager with empty bytes
		_byteManager.SetByteArray();

		// Add members in whatever order you want the data layout to be in
		_byteManager.AddInt(_sampleInt);
		_byteManager.AddFloat(_sampleFloat);
		_byteManager.AddString(_sampleString);

		// Convert byte manager data to a byte array
		return _byteManager.GetByteArray();
	}
}
```

#### IByteManager
This defines the storable data types that are available and provides handy methods for creating a byte array.

#### IDataHandler
This interface allows you to convert the data into a string that can be written to a file. The included implementation converts it to a JSON string, however you can create any type of handler for your data that you need.

#### IEncrypt
The final step before writing the data to a file. If encryption is turned on, you can select the type of encryption by using the dropdown in the Game Save object.