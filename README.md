# 2weistein-Speedrun-Mod
A mod that makes it easier to keep track of in-game times

TODO: Harmony injection modding

## New Features

* Changed Version Identifier
* Enabled Dev Keyboard Commands:
  - `1` Swap Gender
  - `2` Show FPS (Changed to show time in level)
  - `5` Screenshot
  - `7` Heal, Refresh Magic, 9999 Coins, 666 Points
  - `8` Swap Language
  - `9` Unlock All Levels
  - `F12` Skip Level ("Force Finish")
* Added a `/Data/SaveGames/[profile]_run.ini` file for each profile that records the level times
* Removed Bonus Dialog
* Run Comparison
  - The game will look for a "best.ini" in the same format as other run files, with an additional `[Total]` key
  - You will have to maintain the file yourself, so that `F12` level finishes are excluded
* Added Sub-seconds to stats dialog
* Swapped distance travelled to total run time on stats dialog
* Fixed Spinning Camera Bug (Removes Ability to control the camera X-Axis with a controller
* Added stats dialog to final boss level

## Planned Features


* 1080p Resolution Fix

## Installation

Go to the [Releases](https://github.com/wulkanat/2weistein-Speedrun-Mod/releases) and download the latest version.
Then use the included DLLs to replace the files in the `Data` folder of your game's directory.

Game Version Support
| Version | Support | Notes |
| ------- | ------- | ----- |
| `Neuauflage DE v3.0 Build 144` | <ul><li>- [x] </li></ul> |  |
| `Neuauflage EN v3.0 Build 144` | <ul><li>- [x] </li></ul> | Untested |
| `Standard DE v1.0 Build 077` | <ul><li>- [ ] </li></ul> | Planned |
| `Power DE` | <ul><li>- [ ] </li></ul> | Maybe |
| Other | <ul><li>- [ ] </li></ul> | On Request |

## Changes

The following modifications have been made to the game's code

### `BuildSettings`

```cs
public static bool isReleaseVersion = false; // enables dev commands
public static string buildNumber = "114 Speedrun";
```

### `InitScene`
```cs
public void Update()
{
    if (InputEx.GetKeyDown("f12") && !this.isMiniGame)
    {
        // we use this as an indicator that the level was skipped
        SceneManagerState.isMiniGame = true;
        SceneManagerState.Load();
```
### `SceneManager`

```cs
public void Load()
{
  GameState.currentState = States.gui;
  if (!SceneManagerState.currentLevel.isCutScene) // remove endboss check
  // save time until level finished, but only if the level hasn't been skipped
    if (!SceneManagerState.isMiniGame)
    {
      INIFile inifile = new INIFile(Application.dataPath + "/SaveGames/" + Config.profileName + "_run.ini");
      float current = Time.timeSinceLevelLoad + LevelStatistic.addTime;
      float totalTime = current;
      foreach (object obj in inifile.GetSectionList())
      {
        string entry = (string)obj;
        totalTime += float.Parse(inifile.ReadValue(entry, "Time", "0"));
      }
      inifile.WriteValue(SceneManagerState.currentLevel.key, "Time", current.ToString());
      inifile.WriteValue(SceneManagerState.currentLevel.key, "Total", totalTime.ToString());
      inifile.Save(Application.dataPath + "/SaveGames/" + Config.profileName + "_run.ini");
    }
    else
    {
      SceneManagerState.isMiniGame = false;
    }
    // skip bonus dialog
    this.dialog = (GameObject)UnityEngine.Object.Instantiate(Resources.Load("GUI/LevelStatistic/LevelStatisticDialog", typeof(GameObject)), new Vector3(0.5f, 0.5f, 1f), Quaternion.identity);
```
### `LevelStatisticHandler`

```cs
public void Start()
{
  ...
  else if (num3 == 6)
  {
    float current = Time.timeSinceLevelLoad + LevelStatistic.addTime;
    INIFile bestfile = new INIFile(Application.dataPath + "/SaveGames/best.ini");
    float delta = current - float.Parse(bestfile.ReadValue(SceneManagerState.currentLevel.key, "Time", "999"));
    text = string.Concat(new string[]
    {
      this.FormatTime(current, true),
      " (",
      (delta > 0f) ? "+" : "-",
      this.FormatTime(Math.Abs(delta), true),
      ")"
    });
    INIFile bestFile = new INIFile(Application.dataPath + "/SaveGames/best.ini");
    if (delta < 0f)
    {
      bestFile.WriteValue(SceneManagerState.currentLevel.key, "Time", current.ToString());
      bestFile.Save(Application.dataPath + "/SaveGames/best.ini");
    }
  }
  else if (num3 == 7)
  {
    // swap distance travelled to total time took
    INIFile iNIFile = new INIFile(Application.dataPath + "/SaveGames/" + Config.profileName + "_run.ini");
    float total = float.Parse(iNIFile.ReadValue(SceneManagerState.currentLevel.key, "Total", "999"));
    float delta2 = total - float.Parse(new INIFile(Application.dataPath + "/SaveGames/best_full.ini").ReadValue(SceneManagerState.currentLevel.key, "Total", "999"));
    text = string.Concat(new string[]
    {
      this.FormatTime(total, true),
      " (",
      (delta2 > 0f) ? "+" : "-",
      this.FormatTime(Math.Abs(delta2), true),
      ")"
    });
    INIFile bestFile2 = new INIFile(Application.dataPath + "/SaveGames/best_full.ini");
    if (delta2 < 0f && SceneManagerState.currentLevel.key == "19_GodronEndBoss")
    {
      // save the complete run
      iNIFile.Save(Application.dataPath + "/SaveGames/best_full.ini");
    }
```
### `FrameCount`

```cs
// replaced complete function function
public void Update()
{
    this.fps.text = "Time: " + Time.timeSinceLevelLoad.ToString().PadLeft(7);
    this.bg1.text = this.fps.text;
    this.bg2.text = this.fps.text;
}
```
### `InputEx`
```cs
public static float GetAxis(string axis)
{
    ...
    else if (axis == "Mouse X")
    {
        num = Input.GetAxis(axis);
        // remove controller controls
        num2 *= Mathf.Abs(num2);
        InputEx.gamepadEvent = (num2 != 0f);
        num += num2;
    }
    ...
}
```
