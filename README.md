XML should be placed into PnFMods/ModsInstaller_X_X/mods folder.
The file structure follows:

```xml
<code>
	<check name="mod_name" version="mod_version" dev="true_OR_false" debug="debug_logging_level"/>

	<target_File file="file_name1">
		<root_Node>
			<path>
				<action/>
				<action/>

				<path>
					<action/>
					<action/>
				</path>

				<action/>

				<path>
					<action/>
					<action/>
				</path>
				<attrs>
					....
				</attrs>
			</path>
			<attrs>
				....
			</attrs>
		</root_Node>

		<root_Node>
			<path>
				<action/>
			</path>
		</root_Node>
	</target_File>

	<target_File file="имя_файла2">
		<root_Node>
			<path>
				<action/>
				<action/>

				<path>
					<action/>
					<action/>
				</path>

				<action/>

				<path>
					<action/>
					<action/>
				</path>
			</path>
		</root_Node>

		<root_Node>
			<path>
				<action/>
			</path>
		</root_Node>
	</target_File>
</code>
```

All file paths are relative to res_mods.

# 1. Check

Element `<check>` is mandatory. It checks res_mods/installed_mods.xml for its current status.
If element exists and versions are equal then it considers that mod has been already installed and there will be to action taken.
If such element exists but has a lower version then the mod will be reinstalled.

Example:
```xml
<check name="campaigns_in_battle" version="1.0"/>
```
searches installed_mods.xml for a block
```xml
<mod name="campaigns_in_battle" version="1.0"/>
```
When mod installation succeeds then this element will be automatically added to the installed_mods.xml.
When you debug a mod you may add attribute dev="true", this will disable version check and the mod will be installed.

Example:
```xml
<check name="campaigns_in_battle" version="1.0" dev="true"/>
```
There are four debug levels from `debug="0"` to `debug="3"`. Default value for debug is 0, when there is no `dev="true"` and is 3 otherwise.

In case the mode depends on others you may specify them in the ```<check>``` element and execute parts of your mod depending on other mods installation status.

This instruction looks like:
```xml
<name file="string"/>
```
where
- "name" - mod name to be referenced in the future checks.
- "string" - sub string to lookup in the file "uss_settings.xml". A mod is considered installed in case the sub string is found.

Example:
```xml
<check name="campaigns_in_battle" version="1.0">
	<loading_screen file="Roslich_loading_screen.xml"/>
</check>
```
This code will search for "Roslich_loading_screen.xml" sub string in the file "uss_settings.xml".
In this particular case the string  `<xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile>` will match because it contains "Roslich_loading_screen.xml".

After the mod install the information about installation of the mods which are depended on is saved as well.
If during the check it turns out that the mod is installed, but there is a mod was added on which it depends then whole mod is reinstalled.

So at the start the installation we find out the status of the mod and mod on which it depends.
If mod is not installed or there is older version present then installation is started.
If mod is installed and version is ok but a mod was added on which it depends then installation is started as well.

During the installation process executes all code with no attributes `<do_if_mod_installed/>` and `<do_if_mod_not_installed/>` (details will follow).
Code in the `<do_if_mod_installed mod="a mod name the the check" />` element is executed as well in case the specified mod is installed.
And the code in the  `<do_if_mod_not_installed/>` is executed if specified mod is not installed.

# 2. Target_File
```xml
<target_File file="file_name">
	<root_Node>
		...
	</root_Node>
</target_File>
```
Contains actions to be applied to the file "file_name"

The attribute `orig="true"` in `<target_File>` element will force actions on the original file the the game client ignoring existing modified one.
The attribute `clear="true"` , all the changes should be applied to an empty file only. Existing will not be opened for changes or unpacked from the game client.

Example:
```xml
<target_File file="gui/uss_settings.xml">
	<root_Node>
		...
	</root_Node>
</target_File>
<target_File file="gui/unbound/mods/Roslich_loading_screen.xml">
	<root_Node>
		...
	</root_Node>
</target_File>
```
Changes will be applied to the uss_settings.xml and Roslich_loading_screen.xml files.
In case they do not exists they will be created. First we try to open an existing file. In case we fail we will try to unpack the file from the client. If such file does not exist in the client then empty one will be created.
You may use `<target_File file="gui/unbound/mods/Roslich_loading_screen.xml" clear="true">` then we will not try open or unpack file. It will be created as empty. This is useful for the main file of the mod.

# 3. A path element.

`<root_Node>` matches to the root node of a xml file. For markup.xml this is `<ui>`, for battle_elements.xml is `<battle_elements.xml>`.
All `<path>` elements are relative to this one.

In the `<path>` element you may use full block names (for example `<block className="ShipTreeShortElement">`).
If there are a couple identical blocks (with no attributes, or with the same attributes and you need to hit not the first one)
You may specify it's order number in it's parent. Example `<block number="2">`
The search performs only one level deep.

If you need to find a block deeper and there is no desire, or the ability to indicate all the steps of the path, then you can search for a block by its contents (in this case, the search goes to the full depth):
`<find_node tag="" attr_1="" value_1="" strict_1="" attr_2="" value_2="" strict_2="" text="" strict_text="" sub_nodes="" recursive="">`
and
`<find_parent tag="" attr_1="" value_1="" strict_1="" attr_2="" value_2="" strict_2="" text="" strict_text="" sub_nodes="" recursive="" number="">`
where
- `tag=""` - a block tag
- `attr_1=""` - the first attribute name
- `value_1=""` - the first attribute value
- `strict_1=""` - how strict the search is (true - full match, false - sub string search). By default, is false.
- `attr_2=""` - the second attribute name
- `value_2=""` - the second attribute value
- `strict_2=""` - how strict the search is (true - full match, false - sub string search). By default, is false.
- `text=""` - a text node value (for example in uss_settings.xml this would be "../unbound/styles.xml")
- `strict_text=""` - how strict the search is (true - full match, false - sub string search). By default, is false.
- `sub_nodes=""` - depth of the search (true - recursive search in all children, false - only one level deep). By default, true
- `recursive=""` - true, to find all blocks that meet the criteria, false - only the first.

By default, `attr_2="value"`, that is why code
```xml
<find_parent tag="bind" attr_1="name" value_1="instance" attr_2="value" value_2="TaskItemHeader">
```
and
```xml
`<find_parent tag="bind" attr_1="name" value_1="instance" value_2="TaskItemHeader">
```
are the same.

The "tag" attribute is mandatory all others are optional.
This is true for all situations where `tag="" attr_1="" value_1="" attr_2="" value_2="" text="" sub_nodes="" recursive=""` is used.

In the `<find_parent>` you may add `number="NN"` in case you need to go a few levels up.

Example 1:
```xml
<find_node tag="block" attr_1="className" value_1="hint_panel">
```
Will return
```xml
<block className="hint_panel" type="native">
```
Example 2:
```xml
<find_parent tag="bind" attr_1="name" value_1="instance" value_2="TaskItemHeader">
```
Will return a block which contains as a child
```xml
<bind name="instance" value="TaskItemHeader"/>
```
Example 3:
```xml
<find_node tag="bind" attr_1="name" value_1="visible" recursive="true">
```
Returns all the blocks
```xml
<bind name="visible" value="blablabla"/>
```
Example 4:
```xml
<find_parent tag="bind" attr_1="name" value_1="instance" value_2="TaskItemHeader" number="2">
```
Returns a block two levels up from element
```xml
<bind name="instance" value="TaskItemHeader"/>
```
Example 5:
```xml
<find_node tag="width" value_2="240" strict_2="true">
```
Will return
```xml
<width value="240"/> and will skip <width value="240px"/>
```

## 3.1 A path element attributes

You may specify some attributes in the `<attrs>` tag.

Example:
```xml
<a_path>
	<a_path>
		<attrs>
			...
		</attrs>
	</a_path>
	<attrs>
		...
	</attrs>
</a_path>
```
Possible elements inside `<attrs>`:
- `<do_if_mod_installed mod=""/>` - checks if a mod described in `<check>` is installed.
- `<do_if_mod_not_installed mod=""/>` - checks if a mod described in `<check>` is not installed.
- `<do_if_exist tag="" attr_1="" ... />` - checks for a block described in their `tag=""`, `attr_1=""`, etc.
- `<do_if_not_exist tag="" attr_1="" ... />`

Example 1:
```xml
<check name="campaigns_in_battle" version="1.0">
	<loading_screen file="Roslich_loading_screen.xml" />
</check>

<target_File file="gui/unbound/mods/Roslich_loading_screen.xml">
	<root_Node>
		...
		<attrs>
			<do_if_mod_installed mod="loading_screen"/>
		</attrs>
	</root_Node>
</target_File>
```
Will make changes to the file gui/unbound/mods/Roslich_loading_screen.xml only when there is mod  loading_screen described in the `<check>` element is installed.
Example 2:
```xml
<target_File file="gui/unbound/mods/Roslich_loading_screen.xml">
	<root_Node>
		<block className="BattleStats">
			...
			<attrs>
				<do_if_exist tag="block" attr_1="className" value_1="BattleLoading"/>
			</attrs>
		</block>
	</root_Node>
</target_File>
```
Will take actions from `<block className="BattleStats">` only when there is a block `<block className="BattleLoading">` exists in the file being changed.

# 4. Actions

All actions takes place relative to the block on which `<a_path>` has ended.
Possible actions:
- `<remove>` - Blocks removal;
- `<insert>` - Blocks addition;
- `<replace>` - Blocks replacement;
- `<rename>` - Rename attributes in existing blocks;
- `<copy_past>` - Copy or Move some blocks;

All actions execute in the order they appear in the file.

General structure:
```xml
<an_action>
	<some_action_data>
	...
	</some_action_data>
	<attrs>
		<an_attribute/>
		...
		<an_attribute/>
	</attrs>
</an_action>
```

## 4.1 Action attributes

Possible tags inside `<attrs>` are:
- `<log_info value=""/>`
- `<position insert="" tag="" attr_1="" value_1="" .../>`
- `<default_position insert="" tag="" attr_1="" value_1="" .../>`
- `<do_if_exist tag="" attr_1="" value_1="" .../>`
- `<do_if_not_exist tag="" attr_1="" value_1="" .../>`
- `<do_if_mod_installed mod=""/>`
- `<do_if_mod_not_installed/>`
- `<copy_from file="имя_файла" orig=""/>`
- `<rename attr_rename="" old_value="" new_value=""/>`
- `<cut/>`

1. `<log_info/>` outputs a message from it's `value=""` attribute to the python.log file (useful for debug).
2. `<position/>` used by `<insert>` and `<copy_past>` actions to specify action position.
`insert=""` attribute contains a position relative to a block pointed by `tag=""`, `attr_1=""`, and `value_1=""`, etc.
Possible values are:
 - before_node - right before the block
 - after_node - right after the block
 - before_parent - before the parent of the block
 - after_parent - after the parent of the block
 - top - in the beginning of the block
 - no `<position/>` specified - the very end of the block

3. `<default_position/>` is the same as `<position/>` and used in case there is no `<position/>` specified.

 If there is no `<position/>` nor `<default_position/>` in the `<attrs>` element then an error is printed to the log and mod installation is skipped.

4. `<do_if_exist/>` makes the action happen only when there is a block found which is described in this element.

5. `<do_if_not_exist/>` makes the action happen only when there is NO block found which is described in this element.
This might be used to modify uss_settings.xml. Describe a string to be inserted only when it is not in the file yet.

6. `<do_if_mod_installed/>` the action will take place only when specific mod `mod=""` is installed. The `mod=""` attribute should contain mod name from a `<check/>` element.

7. `<do_if_mod_not_installed/>` same as above but mod NON existence is checked.

8. `<copy_from/>` used by `<copy_past>` element as a source file. If there is no `<copy_past>` element then current file is used as a source. If `orig="true"` attribute is present copy action will use original "clean" file from the game client. Otherwise we'll try to read current file and in case of failure we'll try to get from the game client. And if this fail an error will be logged and mod install will be skipped.

9. `<rename/>` is used by `<copy_past>` element to change attribute value of a block being inserted. Where `attr_rename` - source attribute name, `old_value` - an old value, `new_value` - a new value. If there is no `old_value` attribute then attribute will be assigned  `new_value` value.

10. `<cut/>` is used by `<copy_past>` element to move some blocks. Will remove copied source blocks.

`<log_info>` and `<position>` might be specified not only in `<attrs>` element but also as attributes of `<an_action>` element.
Example:
```xml
<an_action>
	...
	<attrs>
		<log_info value="blablabla"/>
		<position insert="" tag="" attr_1="" value_1="".../>
	</attrs>
</an_action>
```
is equal to:
```xml
<an_action log_info="blablabla"  insert="" tag="" attr_1="" value_1=""...>
	...
</an_action>
```
If `<log_info>` and `<position>` are specified in both `<attrs>` and in `<an_action>` then those from `<attrs>` element will take precedence.

## 4.2 Action types

### 4.2.1 Action `<remove tag="" attr_1="" value_1=""...>`

Deletes found block. To delete all found blocks use `recursive="true"`
Example 1:
```xml
<root_Node>
	<block className="AccountLevelBannerWithPromo">
		<remove tag="bind" attr_1="name" value_1="tooltip" value_2="'SlimClientStatusProfileTooltip'"/>
	</block>
</root_Node>
```
Will delete from `<block className="AccountLevelBannerWithPromo">` the string `<bind name="tooltip" value="'SlimClientStatusProfileTooltip'; slimClientData.isFull ? null : {}"/>`

Example 2:
```xml
<root_Node>
	<remove tag="block" attr_1="className" value_1="MinimapShipItem" recursive="true"/>
	<remove tag="block" attr_1="className" value_1="BattleLoading" recursive="true" sub_nodes="false"/>
</root_Node>
```
Will delete all blocks `<block className="MinimapShipItem">` found in whole file, but blocks ` <block className="BattleLoading">` only from `<ui>` element without searching then in sub elements.

Example 3:
```xml
<root_Node>
	<block className="AccountLevelBannerWithPromo">
		<find_parent tag="bind" attr_1="name" value_1="tooltip" value_2="'SlimClientStatusProfileTooltip'">
			<remove/>
		</find_parent>
	</block>
</root_Node>
```
Will delete a whole block which contains `<bind name="tooltip" value="'SlimClientStatusProfileTooltip'; slimClientData.isFull ? null : {}"/>`

### 4.2.2 Action `<insert>`
```xml
<insert>
	<a_block_to_be_inserted_1/>
	...
	<a_block_to_be_inserted_n/>
	<attrs>
		...
	</attrs>
</insert>
```
Inside `<insert>` element there blocks to be added. By the default they are inserted to the end of the block on which a path ends. You may use elements `<position/>` and `<default_position/>` to control this behaviour.
Example:
```xml
<target_File file="gui/uss_settings.xml">
	<root_Node>
		<mods>
			<insert>
				<xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile>
				<swffile>../unbound/flash/Roslich_loading_screen.swf</swffile>
				<attrs>
					<position insert="before_node" attr_1="xmlfile" text="Roslich_icons.xml"/>
					<default_position insert="top"/>
					<do_if_not_exist attr_1="xmlfile" text="../unbound/mods/Roslich_loading_screen.xml"/>
				</attrs>
			</insert>
		</mods>
	</root_Node>
</target_File>
```
If there is a string "../unbound/mods/Roslich_loading_screen.xml" found no action will take place. Otherwise just before `<xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile>` there will be `<insert>` content added: `<xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile>` and `<swffile>../unbound/flash/Roslich_loading_screen.swf</swffile>`
If there is no sub string "Roslich_icons.xml" then insert will be to the beginning of the `<mods>` element.

### 4.2.3 Action `<replace>`
```xml
<replace>
	<old tag="" attr_1="" value_1="".../>
	...
	<old tag="" attr_1="" value_1="".../>
	<new>
		<a_block_to_be_inserted_1/>
		...
		<a_block_to_be_inserted_n/>
	</new>
	<attrs>
		...
	</attrs>
</replace>
```
Element `<replace>` contains elements `<old>` and `<new>`.
`<old>` enumerates blocks to be replaced and `<new>` contains replacement.

Example:
```xml
<root_Node>
	<block className="ShipTreeElement">
		<replace>
			<old tag="bind" attr_1="name" value_1="action" value_2="'left_click'"/>
			<old tag="bind" attr_1="name" value_1="menu" value_2="'ShipTreeMenu'"/>
			<new>
				<bind name="action" value="'left_click';  'selectShipUpgrade' ; {shipId : shipId}"/>
				<bind name="menu" value="'ShipTreeMenu'; {shipId: shipId}"/>
			</new>
		</replace>
	</block>
</root_Node>
```
In block `<block className="ShipTreeElement">` we search for:
`<bind name="action" value="'left_click'*"/>` and
`<bind name="menu" value="'ShipTreeMenu'*"/>`
And then they are replaced by
`<bind name="action" value="'left_click';  'selectShipUpgrade' ; {shipId : shipId}"/>` and
`<bind name="menu" value="'ShipTreeMenu'; {shipId: shipId}"/>`
The number of sub elements of `<old>` and `<new>` might not be the same. Each `<old>` will be replaced with `<new>` contents.

When all elements need to be replaced including sub elements you may use attribute `recursive="true"`.

Example:
```xml
<root_Node>
	<block className="SimpleUIListTeamResultRowRendererLeft">
		<replace>
			<old tag="height" value_2="28px" recursive="true"/>
			<new>
				<height value="30px"/>
			</new>
		</replace>
	</block>
</root_Node>
```
Will replace all `<height value="28px"/>` in the block `<block className="SimpleUIListTeamResultRowRendererLeft">` to `<height value="30px"/>`

### 4.2.4 Action `<copy_past>`
```xml
<root_Node>
	<a_main_path>
		<copy_past>
			<copy>
				<a_path_1>
					<a_path_2/>
				</a_path_1>
				<a_path_3>
					<a_path_4/>
				</a_path_3>
			</copy>
			<attrs>
				...
			</attrs>
		</copy_past>
	</a_main_path>
</root_Node>
```
In the `<copy>` element there are path (or paths) to a block (or blocks) which will be copied. Same rules applied as to `<a_main_path>` taking `<copy>` element as a root block instead of `<root_Node>`. All copied content will be inserted into `<a_main_path>` element taking into account a `<position>` if there is one presents.

Example 1:
```xml
<root_Node>
	<block className="AccountLevelShortBanner">
		<copy_past>
			<copy>
				<block className="AccountLevelBanner">
					<find_node tag="block" attr_1="className" value_1="mc_blurmap_medium"/>
				</block>
			</copy>
			<attrs>
				<position insert="before_node" tag="bind" attr_1="name" value_1="catch" value_2="'onLostTop'" />
			</attrs>
		</copy_past>
	</block>
</root_Node>
```
Will copy contents of  `<block className="AccountLevelBanner">`
```xml
<block className="mc_blurmap_medium" type="native">
	<bind name="appear" value="'startShow'; 0.3; 0; {alpha: 0}; {alpha: 1}"/>
	<bind name="appear" value="'startHide'; 0.15; 0; {alpha: 1}; {alpha: 0}"/>
	<styleClass value="$FullsizeAbsolute"/>
	<bind name="blurMap" value="0"/>
</block>
```
to the `<block className="AccountLevelShortBanner">` element right before `<bind name="catch" value="'onLostTop'; { isOnTop: false }"/>`

Example 2:
```xml
<root_Node>
	<copy_past>
		<copy>
			<block className="AccountLevelBanner">
				<find_node tag="block" attr_1="className" value_1="ShipExtendedTooltip"/>
			</block>
		</copy>
		<attrs>
			<position insert="top" />
			<cut/>
		</attrs>
	</copy_past>
</root_Node>
```
Will cut `<block className="ShipExtendedTooltip">` and insert it into the beginning of the file.

Example 3:
```xml
<root_Node>
	<block className="AccountLevelShortBanner">
		<copy_past>
			<copy>
				<block className="AccountLevelBanner">
					<find_parent tag="bind" attr_1="name" value_1="tooltip" value_2="PromoBannerTooltip"/>
				</block>
			</copy>
			<attrs>
				<position insert="before" tag="bind" attr_1="name" value_1="catch" value_2="'onLostTop'"/>
			</attrs>
		</copy_past>
	</block>
</root_Node>
```
Will find element `<block className="AccountLevelBanner">`
which contains `<bind name="tooltip" value="'PromoBannerTooltip'; isShowPromoRewardBanner ? {} : null"/>` and then will insert `<block className="AccountLevelShortBanner">` before `<bind name="catch" value="'onLostTop'; { isOnTop: false }"/>`

Example 4:
```xml
<target_File file="gui/unbound/mods/Roslich_loading_screen.xml" clear="true">
	<root_Node>
		<copy_past>
			<copy>
				<block className="BattleLoading"/>
				<block className="BattleStats"/>
				<block className="TeamBattleLoadingRightPanel"/>
				<block className="TeamBattlePage"/>
				<block className="UIlistTeamStructureRightTeamsRepeater"/>
			</copy>
			<attrs>
				<copy_from file="gui/unbound/markup.xml" orig="true"/>
				<rename attr_rename="className" old_value="TeamBattlePage" new_value="R_TeamBattlePage"/>
			</attrs>
		</copy_past>
	</root_Node>
</target_File>
```
Will copy five blocks from the original markup.xml enumerated in the `<copy>` and will insert them into the end of the file "gui/unbound/mods/Roslich_loading_screen.xml".
Also the element `<block className="BattleStats"/>` will be renamed to `<block className="R_TeamBattlePage"/>`

### 4.2.5 Action `<rename>`
```xml
<rename tag="" attr_1="" value_1="".../>
```
Searches for (`tag="" attr_1="" value_1=""...`) and replaces the value of the attribute `attr_rename=""` an `old_value=""` to the `new_value=""`.
If `old_value` if omitted then `attr_rename=""` attribute will be assigned with value of `new_value=""`.
If element (`tag="" attr_1="" value_1=""...`) has no attribute `attr_rename=""` then such attribute will be added.

Example 1:
```xml
<root_Node>
	<block className="TeamBattlePage">
		<rename tag="bind" attr_1="name" value_1="instance" value_2="'421px'"  attr_rename="value" old_value="'421px'" new_value="'100%'" recursive="true"/>
	</block>
</root_Node>
```
Searches `<block className="BattleLoading">` element for sub elements `<bind name="instance" value="blablabla _width: '421px' blablabla">` which have string `421px` in the `value` attribute and changes `421px` to `100%`.

as a result
```xml
<bind name="instance" value="'UIlistTeamStructureHeaderLeft'; {  _width: '421px', _isBattleStats: _isBattleStats, _allyPlayerEntityId: allies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator}"/>
```
and
```xml
<bind name="instance" value="'UIlistTeamStructureHeaderRight'; { _width: '421px', _isBattleStats: _isBattleStats, _enemyPlayerEntityId: enemies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator }"/>
```
will become
```xml
<bind name="instance" value="'UIlistTeamStructureHeaderLeft'; {  _width: '100%', _isBattleStats: _isBattleStats, _allyPlayerEntityId: allies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator}"/>
<bind name="instance" value="'UIlistTeamStructureHeaderRight'; { _width: '100%', _isBattleStats: _isBattleStats, _enemyPlayerEntityId: enemies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator }"/>
```
Example 2:
```xml
<root_Node>
	<rename tag="element" attr_1="name" value_1="unboundShipsList" attr_rename="enabled" new_value="false"/>
</root_Node>
```
will transform
```xml
<element name="unboundShipsList" class="lesta.unbound2.UbElement" elementName="HeaderShipList" url="battle_stats.swf" autoPerfTestGroup="header">
```
into
```xml
<element name="unboundShipsList" class="lesta.unbound2.UbElement" elementName="HeaderShipList" url="battle_stats.swf" autoPerfTestGroup="header" enabled="false">
```
