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
When you debug a mod you may add attribute dev="true", this will disble version check and the mod will be installed.

Example:
```xml
<check name="campaigns_in_battle" version="1.0" dev="true"/>
```
There are four debug levels from `debug="0"` to `debug="3"`. Default value for the debug is 0, when there is no `dev="true"` and is 3 otherwise.

In case the mode depends on others you may specify them in the ```<check>``` element and execute parts of your mod depending on other mods installation status.

This instruction looks like:
```xml
<name file="string"/>
```
where
- "name" - mod name to be referenced in the future checks.
- "string" - substring to lookup in the file "uss_settings.xml". A mod is consered installed in case the substring is found.

Example:
```xml
<check name="campaigns_in_battle" version="1.0">
	<loading_screen file="Roslich_loading_screen.xml"/>
</check>
```
This code will search for "Roslich_loading_screen.xml" substring in the file "uss_settings.xml".
In this particular case the string  `<xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile>` will match because it contains "Roslich_loading_screen.xml".

After the mod install the information about installation of the mods which are depended is saved as well.
If during the check it turns out that the mod is installed, but there is a mod was added on which it depends then whole mod is reinstalled.

So at the start the installation we find out the status of the mod and mod on which it depeneds.
If mod is not installed or there is older version present then installation is started.
If mod is installed and version is ok but a mod was added on which it depeds then installation is started as well.

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
In case they do not exists they will be created. First we try to open ann existing file. In case we fail we will try to unpack the file from the client. If such file does not exists in the client then empty one will be created.
You may use `<target_File file="gui/unbound/mods/Roslich_loading_screen.xml" clear="true">` then we will not try open or unpack file. It will be created as empty. This is useful for the main file of the mod.

# 3. A path element

`<root_Node>` matches to the root node of an xml file. For markup.xml this is `<ui>`, for battle_elements.xml is `<battle_elements.xml>`.
All `<path>` elements are reletive to this one.

In the `<path>` element you may use full block names (for example `<block className="ShipTreeShortElement">`).
If there are a couple identical blocks (with no attributes, or with the same attributes and you need to hit not the first one)
You may specify it's order number in it's parent. Example `<block number="2">`
The search performs onle one level deep.

If you need to find a block deeper and there is no desire, or the ability to indicate all the steps of the path, then you can search for a block by its contents (in this case, the search goes to the full depth):
`<find_node tag="" attr_1="" value_1="" strict_1="" attr_2="" value_2="" strict_2="" text="" strict_text="" sub_nodes="" recursive="">`
and
`<find_parent tag="" attr_1="" value_1="" strict_1="" attr_2="" value_2="" strict_2="" text="" strict_text="" sub_nodes="" recursive="" number="">`
where
- `tag=""` - a block tag
- `attr_1=""` - the first attribute name
- `value_1=""` - the first attribute value
- `strict_1=""` - how strict the search is (true - full match, false - substring search). By default is false.
- `attr_2=""` - the second attribute name
- `value_2=""` - the second attribute value
- `strict_2=""` - how strict the search is (true - full match, false - substring search). By default is false.
- `text=""` - a text node value (for example in uss_settings.xml this would be "../unbound/styles.xml")
- `strict_text=""` - how strict the search is (true - full match, false - substring search). By default is false.
- `sub_nodes=""` - depth of the search (true - recursive search in all children, false - only one level deep). By default true
- `recursive=""` - true, to find all blocks that meet the criteria, false - only the first.

By default `attr_2="value"`, that is why code
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

		На выполнение <путь> влияют атрибуты, указанные в его <attrs>. Выглядит это следующим образом:
		
		<путь>
			<путь>
				<attrs>
					...
				</attrs>
			</путь>
			<attrs>
				...
			</attrs>
		</путь>
		
		Внутри <attrs> могут быть:
		<do_if_mod_installed mod=""/>
		<do_if_mod_not_installed mod=""/>
		<do_if_exist tag="" attr_1="" ... />
		<do_if_not_exist tag="" attr_1="" ... />
		
		<do_if_mod_installed/> и <do_if_mod_not_installed/> проверяют, установлен ли мод, описанный в <check>
		<do_if_exist> и <do_if_not_exist> проверяют наличие блока, описанного в их tag="" attr_1="" ...
		
		Например:
		<check name="campaigns_in_battle" version="1.0">
			<loading_screen file="Roslich_loading_screen.xml">
		</check>
		
		<target_File file="gui/unbound/mods/Roslich_loading_screen.xml">
			<root_Node>
				...
				<attrs>
					<do_if_mod_installed mod="loading_screen"/>
				</attrs>
			</root_Node>
		</target_File>	
		Внесёт изменения в файл gui/unbound/mods/Roslich_loading_screen.xml, только если установлен мод loading_screen, описанный в блоке <check>

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
		Будет выполнять действия в <block className="BattleStats"> , только если в редактируемом файле существует <block className="BattleLoading">

4.	Действия
		
		Все действия ведутся относительно того блока, на котором в данный момент закончился <путь>

		Возможные действия:
		<remove>		Удаление блоков
		<insert>		Вставка блоков
		<replace>		Замена блоков
		<rename>		Переименование атрибутов существующих блоков
		<copy_past>		Копирование, или перенос блоков
		Действия выполняются в том порядке, в котором они находятся в файле.

		Структура действий:
		<действие>
			<данные_действия/>
			...
			<данные_действия/>
			<attrs>
				<атрибут/>
				...
				<атрибут/>
			</attrs>
		</действие>

4.1		Атрибуты действия

		Внутри <attrs> можно перечислить следующие атрибуты:
		<log_info value=""/>
		<position insert="" tag="" attr_1="" value_1="" .../>
		<default_position insert="" tag="" attr_1="" value_1="" .../>
		<do_if_exist tag="" attr_1="" value_1="" .../>
		<do_if_not_exist tag="" attr_1="" value_1="" .../>
		<do_if_mod_installed mod=""/>
		<do_if_mod_not_installed/>
		<copy_from file="имя_файла" orig=""/>
		<rename attr_rename="" old_value="" new_value=""/>
		<cut/>
		
		<log_info/>
		выводит в python.log сообщение, указанное в value="", во время выполнения этого <действие>

		<position/>
		используется в <insert> и <copy_past> для указания места вставки.
			в insert="" указывается позиция относительно найденного блока (tag="" attr_1="" value_1="" ...)
			insert может быть:
			before_node		перед найденным блоком
			after_node		после найденного блока
			before_parent	перед блоком, содержащим найденный блок
			after_parent	после блока, содержащего найденный блок
			top				в начало блока, на котором закончился путь
			если <position/> нет, то вставка идёт в конец блока, на котором закончился путь.

		<default_position/>
		используется там же, для указания места вставки, если <position/> не найдено.
		Если не найдено ни <position/>, ни <default_position/>, то в лог выводится ошибка, и установка мода пропускается.

		<do_if_exist/>
		указывает, что это <действие> надо выполнять только в том случае, если найден блок, описанный этим атрибутом.

		<do_if_not_exist/>
		указывает, что это <действие> надо выполнять только в том случае, если не найден блок, описанный этим атрибутом.
		Например можно использовать для того, чтобы вносить изменения в uss_settings.xml. Описываем в атрибуте строку,
		которую собираемся внести в файл и тогда она будет вноситься только в том случае, если её ещё нет.

		<do_if_mod_installed/>
		указывает, что это <действие> надо выполнять только в том случае, если мод mod="" установлен.
		В mod="" указывается имя мода, которое было задано в <check/>

		<do_if_mod_not_installed/>
		указывает, что это <действие> надо выполнять только в том случае, если мод mod="" не установлен.
		В mod="" указывается имя мода, которое было задано в <check/>

		<copy_from/>
		используется в <copy_past>, для указания файла, из которого происходит копирование.
		если <copy_from> нет, то копирование идёт из текущего файла. Если orig="true", то копирование будет произведено
		из "чистого", распакованного из клиента файла. Если orig нет, то идёт попытка прочитать существующий файл,
		если не получается, пытаемся распаковать из клиента. Если не получилось - ошибка в лог и пропуск установки мода.

		<rename/>
		используется в <copy_past>, для переименовывания значения атрибута вставляемого блока
		где attr_rename - имя переименовываемого атрибута, old_value - старое значение, new_value - новое значение.
		Если не задавать old_value, то атрибуту будет присвоено значение new_value.

		<cut/>
		используется в <copy_past> в том случае, если надо не скопировать, а перенести блоки. Удаляет скопированный блок в источнике.

		<log_info> и <position> можно указывать не в <attrs>, а напрямую в атрибутах <действие>
		таким образом записи
		<действие>
			...
			<attrs>
				<log_info value="blablabla"/>
				<position insert="" tag="" attr_1="" value_1="".../>
			</attrs>
		</действие>
		и
		<действие log_info="blablabla"  insert="" tag="" attr_1="" value_1=""...>
			...
		</действие>
		равнозначны. Если <log_info> и <position> указаны одновременно и в <attrs> и в <действие>, приоритет будет у <attrs>

4.2		Виды действий

4.2.1	Действие <remove tag="" attr_1="" value_1=""...>

		удаляет найденный блок. Если надо удалить все найденные блоки, то указывается recursive="true"
		Например:
		<root_Node>
			<block className="AccountLevelBannerWithPromo">
				<remove tag="bind" attr_1="name" value_1="tooltip" value_2="'SlimClientStatusProfileTooltip'"/>
			</block>
		</root_Node>
		удалит из <block className="AccountLevelBannerWithPromo">
		строку <bind name="tooltip" value="'SlimClientStatusProfileTooltip'; slimClientData.isFull ? null : {}"/>

		Такой вариант:
		<root_Node>
			<remove tag="block" attr_1="className" value_1="MinimapShipItem" recursive="true"/>
			<remove tag="block" attr_1="className" value_1="BattleLoading" recursive="true" sub_nodes="false"/>
		</root_Node>
		удалит все блоки <block className="MinimapShipItem"> найденные во всём файле, а блоки <block className="BattleLoading">
		только непосредственно из <ui/> и не будет искать их во вложенных блоках.

		<root_Node>
			<block className="AccountLevelBannerWithPromo">
				<find_parent tag="bind" attr_1="name" value_1="tooltip" value_2="'SlimClientStatusProfileTooltip'">
					<remove/>
				</find_parent>
			</block>
		</root_Node>
		удалит целиком блок, содержащий <bind name="tooltip" value="'SlimClientStatusProfileTooltip'; slimClientData.isFull ? null : {}"/>

4.2.2	Действие <insert>

		<insert>
			<вставляемые_блоки/>
			...
			<вставляемые_блоки/>
			<attrs>
				...
			</attrs>
		</insert>
		Внутри <insert> находятся блоки, которые необходимо добавить. По умолчанию они добавляются в конец блока, на котором закончился путь.
		Для уточнения места, куда надо добавлять блоки, используется атрибуты <position/> и <default_position/>
		Например:
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
		если в файле будет найдена строка, содержащая "../unbound/mods/Roslich_loading_screen.xml", то ничего вставляться не будет.
		Если строки нет, то перед <xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile> добавится содержимое <insert>,
		то есть <xmlfile>../unbound/mods/Roslich_loading_screen.xml</xmlfile> и <swffile>../unbound/flash/Roslich_loading_screen.swf</swffile>
		Если не будет найдена строка с текстом "Roslich_icons.xml", то вставка произойдёт в начало блока <mods>

4.2.3	Действие <replace>

		<replace>
			<old tag="" attr_1="" value_1="".../>
			...
			<old tag="" attr_1="" value_1="".../>
			<new>
				<вставляемые_блоки/>
				...
				<вставляемые_блоки/>
			</new>
			<attrs>
				...
			</attrs>
		</replace>
		Внутри <replace> находятся строки <old> и блок <new>
		В <old> перечисляются блоки, которые надо заменить.
		Внутри <new> - чем заменить.
		Например:
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

		В блоке <block className="ShipTreeElement"> ищутся
		<bind name="action" value="'left_click'*"/>
		<bind name="menu" value="'ShipTreeMenu'*"/>
		и меняются на
		<bind name="action" value="'left_click';  'selectShipUpgrade' ; {shipId : shipId}"/>
		<bind name="menu" value="'ShipTreeMenu'; {shipId: shipId}"/>
		Количество <old> и блоков внутри <new> не обязательно должны совпадать. Каждая <old> заменится на содержимое <new>

		Если необходимо заменить не одну, а все найденные строки, то можно использовать атрибут recursive.
		Например:
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
		Заменит все <height value="28px"/>, найденные в <block className="SimpleUIListTeamResultRowRendererLeft">, на <height value="30px"/>


4.2.4	Действие <copy_past>

		<root_Node>
			<основной_путь>
				<copy_past>
					<copy>
						<путь>
							<путь/>
						</путь>
						<путь>
							<путь/>
						</путь>
					</copy>
					<attrs>
						...
					</attrs>
				</copy_past>
			</основной_путь>
		</root_Node>

		Внутри <copy> указывается путь (либо пути) к блоку (блокам), которые надо скопировать. Правила те же, что и для <основной_путь>,
		за исключением того, что корневой блок не <root_Node>, а <copy>.
		Всё скопированное будет вставлено в <основной_путь>, с поправкой на <position>, если она есть.
		Например:
		<root_Node>
			<block className="AccountLevelShortBanner">
				<copy_past>
					<copy>
						<block className="AccountLevelBanner">
							<find_node tag="block" attr_1="className" value_1="mc_blurmap_medium"/>
						</block>
					</copy>
					<attrs>
						<position insert="before_node" tag="bind" attr_1="name" value_1="catch" value_2="'onLostTop'">
					</attrs>
				</copy_past>
			</block>
		</root_Node>
		Скопирует из <block className="AccountLevelBanner">
		<block className="mc_blurmap_medium" type="native">
			<bind name="appear" value="'startShow'; 0.3; 0; {alpha: 0}; {alpha: 1}"/>
			<bind name="appear" value="'startHide'; 0.15; 0; {alpha: 1}; {alpha: 0}"/>
			<styleClass value="$FullsizeAbsolute"/>
			<bind name="blurMap" value="0"/>
		</block>
		и вставит его в <block className="AccountLevelShortBanner">, перед <bind name="catch" value="'onLostTop'; { isOnTop: false }"/>

		А такой вариант:
		<root_Node>
			<copy_past>
				<copy>
					<block className="AccountLevelBanner">
						<find_node tag="block" attr_1="className" value_1="ShipExtendedTooltip"/>
					</block>
				</copy>
				<attrs>
					<position insert="top">
					<cut/>
				</attrs>
			</copy_past>
		</root_Node>
		вырежет блок <block className="ShipExtendedTooltip"> и вставит его в начало файла.

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
		найдёт в <block className="AccountLevelBanner">
		блок, содержащий <bind name="tooltip" value="'PromoBannerTooltip'; isShowPromoRewardBanner ? {} : null"/>
		и вставит его в <block className="AccountLevelShortBanner">, перед <bind name="catch" value="'onLostTop'; { isOnTop: false }"/>

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
			<root_Node>
		</target_File>
		Скопирует из оригинального markup.xml пять блоков, перечисленных в <copy>,
		вставит их в конец файла "gui/unbound/mods/Roslich_loading_screen.xml"
		и переименует <block className="BattleStats"/> в <block className="R_TeamBattlePage"/>

4.2.5	Действие <rename>

		<rename tag="" attr_1="" value_1="".../>
		Ищет блок (tag="" attr_1="" value_1=""...) и заменяет в значении attr_rename, old_value на new_value
		Если old_value не указан, то attr_rename примет значение new_value
		Если у блока (tag="" attr_1="" value_1=""...) нет атрибута attr_rename, то такой атрибут будет создан
		например:
		<root_Node>
			<block className="TeamBattlePage">
				<rename tag="bind" attr_1="name" value_1="instance" value_2="'421px'"  attr_rename="value" old_value="'421px'" new_value="'100%'" recursive="true"/>
			</block>
		</root_Node>
		находит в <block className="BattleLoading"> все блоки <bind name="instance" value="blablabla _width: '421px' blablabla">, содержащие в атрибуте value строку '421px' и меняет в них '421px' на '100%'.
		В итоге, вместо
		<bind name="instance" value="'UIlistTeamStructureHeaderLeft'; {  _width: '421px', _isBattleStats: _isBattleStats, _allyPlayerEntityId: allies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator}"/>
		и
		<bind name="instance" value="'UIlistTeamStructureHeaderRight'; { _width: '421px', _isBattleStats: _isBattleStats, _enemyPlayerEntityId: enemies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator }"/>
		получаем
		<bind name="instance" value="'UIlistTeamStructureHeaderLeft'; {  _width: '100%', _isBattleStats: _isBattleStats, _allyPlayerEntityId: allies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator}"/>
		<bind name="instance" value="'UIlistTeamStructureHeaderRight'; { _width: '100%', _isBattleStats: _isBattleStats, _enemyPlayerEntityId: enemies[0].id,                 _battleType: battleType, _isSpectator: _isSpectator }"/>
		
		А вариант:
		<root_Node>
			<rename tag="element" attr_1="name" value_1="unboundShipsList" attr_rename="enabled" new_value="false"/>
		</root_Node>
		превратит строку
		<element name="unboundShipsList" class="lesta.unbound2.UbElement" elementName="HeaderShipList" url="battle_stats.swf" autoPerfTestGroup="header">
		в
		<element name="unboundShipsList" class="lesta.unbound2.UbElement" elementName="HeaderShipList" url="battle_stats.swf" autoPerfTestGroup="header" enabled="false">
