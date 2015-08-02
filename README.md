# TTT Git Patching
Yesterday night, I accidentally corrupted a Lua file and I had to validate and re-download my entire server. Well, I lost all of the custom TTT changes I've added to TTT over the years, such as scoreboard colors, name change kick immunity, and check idle immunity. Now, this has happened to me multiple times and usually I just add the changes back manually, but this time, I decided to use a Git repository!

This repository serves as an example to anyone else who wants to patch TTT in the same way as me.

# Initialize your repository
Start by initializing your repository in the garrysmod directory of your server. Also create (or copy) the .gitignore and .gitattributes files. Then, add the gamemodes/terrortown directory.

```
cd GarrysModDS/garrysmod
git init
git add .gitignore
git add .gitattributes
git add gamemodes/terrortown
```

Before you commit, get the current version of TTT to put in your commit message:

```
grep -R GM.Version gamemodes
git commit -m "Add terrortown gamemode version 2015-05-25"
```

# Start Branching
Basically, you want your master branch to be the current version of TTT at all times. **Never** commit your custom changes to the master branch.

Here are some examples of the branches I made tonight.

## Scoreboard Colors
- Colors the names of the admin, supermod, moderator, trial, member, and regular groups.

```
git checkout -b scoreboard_colors
git add gamemodes/terrortown/gamemode/vgui/sb_row.lua
git commit -m "Add scoreboard colors"
git format-patch master --stdout > p_scoreboard_teams.patch
git checkout master
```

## Name Change Immunity
- Admins, supermods, and moderators have immunity to name changes

```
git checkout -b namechangekick_immunity
git add gamemodes/terrortown/gamemode/init.lua
git commit -m "Add NameChangeKick immunity"
git format-patch master --stdout > p_namechangekick_immunity.patch
git checkout master
```

## Check Idle Immunity
- Admins and supermods have immunity to idle kicks

```
git checkout -b checkidle_immunity
git add gamemodes/terrortown/gamemode/cl_init.lua
git commit -m "Add CheckIdle immunity"
git format-patch master --stdout > p_checkidle_immunity.patch
git checkout master
```

## Haste Mode Overtime
- Haste Mode counts upwards after it goes into overtime

```
git checkout -b hastemode_overtime
git add gamemodes/terrortown/gamemode/cl_hud.lua
git commit -m "Add Haste Mode overtime"
git format-patch master --stdout > p_hastemode_overtime.patch
git checkout master
```

## FCVAR_USERINFO fixes
- Scripts can read ttt_spectator_mode and ttt_mute_team_check, for instance, not giving pointshop points when idle

```
git checkout -b fcvar_userinfo
git add gamemodes/terrortown/gamemode/cl_help.lua
git commit -m "Add FCVAR_USERINFO"
git format-patch master --stdout > p_fcvar_userinfo.patch
git checkout master
```

## Admin Access
- Admins can use ttt_print_traitors
- Admins can use ttt_print_adminreport
- Admins can use ttt_print_karma
- Admins can use ttt_print_damagelog (during round)

```
git checkout -b admin_access
git add gamemodes/terrortown/gamemode/admin.lua
git commit -m "Add admin access"
git format-patch master --stdout > p_admin_access.patch
git checkout master
```

## Real Disarm
- Planting C4 will show a list of safe and not safe wires
- Owners and Traitors cannot instantly defuse C4

```
git checkout -b real_disarm
git add gamemodes/terrortown/entities/entities/ttt_c4/cl_init.lua
git add gamemodes/terrortown/entities/entities/ttt_c4/shared.lua
git commit -m "Add real disarm"
git format-patch master --stdout > p_real_disarm.patch
```

# Adding patch files to master
Now that I've created a decent number of patch files, I want to commit those patch files to the master so that I do not lose them, and I can use them in my update script and version them:

```
git add *.patch
git commit -m "Add patch files for all current brances"
```

# Updating a committed patch
I'm now making additions to an existing branch, and since I've already committed the patch file, I need to create it outside the Git repository:

```
git checkout scoreboard_colors
git add gamemodes/terrortown/gamemode/vgui/sb_row.lua
git commit -m "Add karma colors"
git format-patch master --stdout > ../p_scoreboard_colors.patch
git checkout -f master
```

After I'm back in master, I can move the patch file into the repository:

```
mv ../p_scoreboard_colors.patch .
git commit -m "Add new version of patch with karma colors"
```

# The update script
This is the script I use to update my Garry's Mod server now:

```
WD=`pwd`
cd /servers
./steam/steamcmd.sh +login anonymous +force_install_dir $GM1/.. +app_update 4020 validate +quit
cd $WD

WD=`pwd`
cd $GM1
git apply p_admin_access.patch
git apply p_checkidle_immunity.patch
git apply p_fcvar_userinfo.patch
git apply p_hastemode_overtime.patch
git apply p_namechangekick_immunity.patch
git apply p_real_disarm.patch
git apply p_scoreboard_colors.patch
cd $WD
```

# Removing all patched changes
Simply do a hard reset to head to remove all patched changes and re-patch.

`git reset --hard HEAD`
