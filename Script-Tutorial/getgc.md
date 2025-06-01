# getgc
* getgc通常用于获取游戏中的表
* 如果用户的表是
```lua
return {
    ["Fuck"] = 20,
    ["Die"] = 91,
    ["Mother"] = 0,
    ["Father"] = 0,
}
```
* 如果我们想要修改表中"Mother"和"Father"的数量为无限时
```lua
for _,v in next, getgc(true) do -- getgc()中的true代表是否获取游戏内部表
    if typeof(v) == "table" then -- 判断是否为表
        if rawget(v, "Mother") and rawget(v, "Father") then
            rawset(v, "Mother", math.huge)
            rawset(v, "Father", math.huge)
            -- 将"Mother"和"Father"改为无限
        end
    end
end
```
