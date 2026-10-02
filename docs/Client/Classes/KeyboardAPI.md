### Class
`RustBuster2016.API.KeyboardAPI`

### Description
A low-level, `SendInput`-based API for synthesizing keyboard/mouse scan-code events, primarily meant for
sending key strokes to DirectX applications that don't respond to normal simulated input (e.g.
`SendKeys`). Based on the DirectInput scan-code list documented at
[gamespp.com](http://www.gamespp.com/directx/directInputKeyboardScanCodes.html).

### `InputType` enum (`[Flags]`)
`Mouse = 0`, `Keyboard = 1`, `Hardware = 2` - which `INPUT` union member `SendKey` fills in.

### `DirectXKeyStrokes` enum
Every DirectInput scan code (`DIK_ESCAPE`, `DIK_1`..`DIK_0`, `DIK_A`..`DIK_Z`, `DIK_F1`..`DIK_F15`,
arrow/numpad keys, `DIK_LEFTMOUSEBUTTON`..`DIK_MOUSEWHEELDOWN`, etc.) as scan code values, not the
`System.Windows.Forms`/`UnityEngine.KeyCode` virtual-key values.

### Members

#### `static void SendKey(DirectXKeyStrokes key, bool KeyUp, InputType inputType)`
Sends a single scan-code key-down (`KeyUp = false`) or key-up (`KeyUp = true`) event via `SendInput`.

```csharp
// Press and release the "E" key.
KeyboardAPI.SendKey(KeyboardAPI.DirectXKeyStrokes.DIK_E, false, KeyboardAPI.InputType.Keyboard);
KeyboardAPI.SendKey(KeyboardAPI.DirectXKeyStrokes.DIK_E, true, KeyboardAPI.InputType.Keyboard);
```

#### `static void SendKey(ushort key, bool KeyUp, InputType inputType)`
Same as above but takes a raw scan code directly, for keys not present in `DirectXKeyStrokes`.

### Notes
- This sends real OS-level input events (like a hardware keyboard would), it does not call into
  Unity's `Input` class - anything in the foreground window will receive it, not just the game.
- `Input`/`InputUnion`/`MouseInput`/`KeyboardInput`/`HardwareInput` are the public P/Invoke interop
  structs backing `SendInput`; plugins normally only need `SendKey`.
