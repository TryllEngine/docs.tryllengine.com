

# Struct FTryllDeliveryMode



[**ClassList**](annotated.md) **>** [**FTryllDeliveryMode**](struct_f_tryll_delivery_mode.md)



[More...](#detailed-description)

* `#include <TryllVoiceProfile.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**Expressiveness**](#variable-expressiveness)   = `-1.f`<br> |
|  FString | [**Instruction**](#variable-instruction)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**ReferenceVoice**](#variable-referencevoice)  <br> |
|  float | [**SpeedScale**](#variable-speedscale)   = `1.f`<br> |












































## Detailed Description


One named delivery (mood or manner of speaking) of a voice. What each field does depends on the TTS family; a field the voice's model cannot use is rejected with InvalidVoiceProfile (3023) when the voice is set. A mode with no fields set behaves like the built-in "neutral" mode, so every profile can define the same names and game code stays portable across models. 


    
## Public Attributes Documentation




### variable Expressiveness 

```C++
float FTryllDeliveryMode::Expressiveness;
```



Chatterbox: exaggeration for this mode, 0.0-2.0. Negative uses the profile's Expressiveness. 


        

<hr>



### variable Instruction 

```C++
FString FTryllDeliveryMode::Instruction;
```



Maya1: sustained delivery clause appended to the voice description. A known keyword (angry, sad, whisper, ...) becomes its tuned clause; anything else is appended as written. 


        

<hr>



### variable Name 

```C++
FString FTryllDeliveryMode::Name;
```



Name passed to Set Delivery Mode: lowercase letters, digits and '\_', 1-32 characters, unique in the profile. "neutral" is built in and cannot be defined. 


        

<hr>



### variable ReferenceVoice 

```C++
FTryllStoragePath FTryllDeliveryMode::ReferenceVoice;
```



Clone families (Pocket TTS, Chatterbox, Qwen3-TTS Base): a clip of this character speaking in this mood, relative to the storage root. Replaces the profile's reference voice while the mode is active. 


        

<hr>



### variable SpeedScale 

```C++
float FTryllDeliveryMode::SpeedScale;
```



Multiplies the profile speed while the mode is active. Must stay 1.0 on models without speed control. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceProfile.h`

