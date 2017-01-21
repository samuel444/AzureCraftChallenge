# Azurecraft Challenge

## Samuel Whitby

This is my submission for the January 2017 Azurecraft Challenge.

##### Code

###### Main

The code in the Main folder is for the main computer. This computer controls the turtles.

This code maks the turtles move:

```
rednet.open("left")
next = 0
monitor = peripheral.wrap("right")
monitor.clear()
monitor.setBackgroundColor(colors.yellow)
monitor.setTextColor(colors.red)
monitor.setTextScale(5)
monitor.setCursorPos(5, 4)
monitor.write("3")
sleep(1)
monitor.setCursorPos(5, 4)
monitor.clear()
monitor.write("2")
sleep(1)
monitor.setCursorPos(5, 4)
monitor.clear()
monitor.write("1")
sleep(1)
monitor.setCursorPos(4, 4)
monitor.clear()
redstone.setOutput("back", true)
monitor.write("GO!")
rednet.broadcast("start")
sleep(1)
monitor.clear()
done = true
monitor.setTextScale(2,5)
finishers = 0
while done do
  timer = textutils.formatTime(os.time(), true)
  local id, message = rednet.receive()
  if id == 23.0 then
    next = 20
  elseif id == 13.0 then
    next = 16
  elseif id == 16.0 then
    next = 12
  elseif id == 6.0 then
    next = 8
  else
    next = 4
  end
  monitor.setCursorPos(next, (message - 1))
  monitor.write(" ")
  monitor.setCursorPos(next, message)
  monitor.write("X")
  if message == 10 then
    redstone.setOutput("back", false)
    finishers = finishers + 1
    if finishers == 1 then
      winner = next / 4
      rednet.send(19, "stop")
    end
    monitor.setCursorPos(next, 12)
    monitor.write(tostring(finishers))
    if finishers == 5 then
      done = false
    end
  end
end
monitor.setCursorPos(1.5, 14)
monitor.write("Congratulations turtle " .. tostring(winner))
monitor.setCursorPos(1.5, 15)
monitor.write("for winning this race")
```

This code makes everything reset and brings the turtles back to the start

...
rednet.open("left")
rednet.broadcast("reset")
monitor = peripheral.wrap("right")
monitor.clear()
redstone.setOutput("back", true)
sleep(9)
redstone.setOutput("back", false)
...


