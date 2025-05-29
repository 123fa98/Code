#  Dump MoonSec V3 Script
## 教程
* 1.打开网络请求 [首页->游戏设置->安全->允许 HTTP 请求]
* 2.在"ServerScriptService"的属性中勾选"LoadStringEnabled"
* 3.在"ServerScriptService"创建一个"Script"
* 4.在创建的脚本中填入"Dump MoonSec V3 Script" [后续会提到]
* 5.右键创建的脚本 [存储/导出->另存为本地插件...-]
* 6.在"测试"界面选择"运行" (如果没有新的窗口出现请把运行方式改为"服务器")

## Dump MoonSec V3 Script
```lua
local RunService = game:GetService("RunService") do
	assert(RunService:IsServer() and RunService:IsStudio(), "MoonSec V3 Dump必须运行在server或studio")
	if not pcall(loadstring, "") then
		error("必须开启loadstring才能Dump MoonSec V3")
	end
end

local DataToCode do
	local DataToCode_request, DataToCode_source = pcall(function()
		return game:GetService("HttpService"):GetAsync("https://raw.githubusercontent.com/78n/Roblox/refs/heads/main/Lua/Libraries/DataToCode/DataToCode.luau")
	end)
	assert(DataToCode_request, '导入"DataToCode"插件错误(尝试保存到本地): ' .. DataToCode_source)

	local CompiledDataToCode = loadstring(DataToCode_source, "DataToCode")
	DataToCode = CompiledDataToCode()
end

local Location = game:GetService("HttpService"):GenerateGUID(false)
local ScriptEditor = game:GetService("ScriptEditorService")
local shared = shared
shared[Location] = DataToCode

function DataToCode.output(tbl)
	shared[Location] = nil
	local Serialized = DataToCode.Convert(tbl, true)
	local DisplayScript = Instance.new("LocalScript", game)
	DisplayScript.Name = "Dumped_"..math.floor(os.clock())

	ScriptEditor:UpdateSourceAsync(DisplayScript, function()
		return Serialized
	end)

	ScriptEditor:OpenScriptDocumentAsync(DisplayScript)
end

local setfenv, error, loadstring, type, info = setfenv, error, loadstring, type, debug.info -- localizing so it doesnt retrieve it from the enviornment
local CClosures = {}

local function newcclosure(func)
	CClosures[func] = "C"

	return func
end

local function islclosure(func)
	return info(func, "l") ~= -1
end

local env = getfenv()

env.setfenv = newcclosure(function(func, ...)
	if type(func) == "function" then -- This is how they check if a function is a CClosure
		error('无法使用"setfenv"设置环境')
	end

	return setfenv(func, ...)
end)

env.debug = (function()
	local newdebug = table.clone(debug) -- They retrieve it weird

	newdebug.getinfo = newcclosure(function(func) -- Encrypted only checks the 'what'
		return {
			what = CClosures[func] or islclosure(func) and "Lua" or "C"
		}
	end)

	return newdebug
end)()

env.loadstring = newcclosure(function(code : string, chunkname : string, ...)
	if type(code) == "string" then
		local Start,End,Pattern,Constants = code:find("([%a_][%w_]*%([%a_][%w_]*%((.)%)%))")

		if Constants then
			local Start, End, Reassign, Variable, full, Constants = code:find("([^%s]([%a_][%w_]*)[%s]*=[%s]*)([%a_][%w_]*%([%a_][%w_]*%((.)%)%)).-return %2%(%.%.%.%)")

			if not Start then
				Start, End, Reassign, Variable, full, Constants = code:find("(([%a_][%w_]*)[%s]*=[%s]*)([%a_][%w_]*%([%a_][%w_]*%((.)%)%)).-return %2%(%.%.%.%)") -- very lazy fix and I'm sorry :)
			end

			code = code:sub(1, Start-1+#Reassign)..`(shared["{Location}"].output({Constants}) or function() end)`..code:sub(Start+#full+3)
		end
	end

	return loadstring(code, chunkname, ...)
end);

-- 把 MoonSec V3 脚本放在下面

```

## 还原源码
* 还原方法为使用AI
* [谷歌AI](aistudio.google.com)
