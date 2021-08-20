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
* Added Sub-seconds to stats dialog
* Swapped distance travelled to total run time on stats dialog
* Added stats dialog to final boss level

## Installation

Go to the [Releases](https://github.com/wulkanat/2weistein-Speedrun-Mod/releases) and download the latest version.
Then use the included DLLs to replace the files in the `Data` folder of your game's directory.

*Note: for now, only the re-released 3.0 version is suppored*

## Changes

The following modifications have been made to the game's code

### `BuildSettings`

```cs
public static bool isReleaseVersion = false; // enables dev commands
public static string buildNumber = "114 Speedrun";
```

### `SceneManager`

```cs
public void Load()
{
    GameState.currentState = States.gui;
    if (!SceneManagerState.currentLevel.isCutScene) // remove endboss check
          // save time until level finished
          INIFile inifile = new INIFile(Application.dataPath + "/SaveGames/" + Config.profileName + "_run.ini");
          inifile.WriteValue(SceneManagerState.currentLevel.key, "Time", (Time.timeSinceLevelLoad + LevelStatistic.addTime).ToString());
          inifile.Save(Application.dataPath + "/SaveGames/" + Config.profileName + "_run.ini");
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
    text = LNG.Get("GLB_STATISTIC_TIME", "Benötigte Zeit: {0}");
    text = string.Format(text, this.FormatTime(unchecked(Time.timeSinceLevelLoad + LevelStatistic.addTime), true)); // changed parameter to `true` to include fractions
  }
  else if (num3 == 7)
  {
    // swap distance travelled to total time took
    text = "Total time: {0}";
    INIFile inifile = new INIFile(Application.dataPath + "/SaveGames/" + Config.profileName + "_run.ini");
    float totalTime = 0;
    foreach (string entry in inifile.GetSectionList())
    {
        totalTime += float.Parse(inifile.ReadValue(entry, "Time", "0"));
    }
    text = string.Format(text, this.FormatTime(totalTime, true));
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
