# Azurecraft Challenge

## Samuel Whitby

This is my submission for the January 2017 Azurecraft Challenge - 3. Advanced Monitor Challenges

You can see the results in action by watching my video on YouTube:

[https://www.youtube.com/watch?v=J09rdhzWC2U](https://www.youtube.com/watch?v=J09rdhzWC2U)

### The Challenge

In this challenge, we want you to race some turtles.  As the turtles are racing you will show the progress of the race on a monitor.  When a turtle wins then it would be nice for the winner to have their name displayed on a large screen. 

* 5 turtles
* The first to advance 10 spaces wins
* Turtles move forward randomly 
* Race to be shown live on a monitor
* Winner to be congratulated on a monitor 

### Extra Credit!

As well as writing code to meet the challenge as above, I have added the following.

* A timer which displays the race time
* A world record is maintained. If the winner finishes faster than the existing record time, they set a new record and a message is shown
* Starting gates which come down when the race begins
* Reset code returns the turtles back to the start, clears monitors and resets the timer ready for a new race

### Code

The code here is in four folders

*Main* is the code for the main computer, which starts and resets races.

*Record Holder* is the code for the finish line computer, which maintains the world record information.

*Timer* is the code for the race timer. 

*Turtles* is the code for the racing turtles. 

#### Main

The code in the Main folder is for the main computer. It runs two programs, `move` and `reset`

`move` sends broadcast messages to the network which the other computers listen for. When a race is in progress, it listens for progress messages sent back from the turtles, which it then displays on the monitor to show each turtle's position in the race. The first turtle to move 10 times is declared the winner. The timer computer is then sent a message so that it stops the timer.

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

```
rednet.open("left")
rednet.broadcast("reset")
monitor = peripheral.wrap("right")
monitor.clear()
redstone.setOutput("back", true)
sleep(9)
redstone.setOutput("back", false)
```

#### Turtle

This is the turtle code which has functions so that when the function is called it will rather start racing or will bring them back to the beginning.

It listens for the **start** message from the main computer, moves forward at random intervals, until it has moved 10 times and then 

```
function race()
  close = 0
  done = false
  while not done do
    wait = math.random(10)
    wait = wait / 10
    os.sleep(wait)
    turtle.forward()
    turtle.forward()
    close = close + 1
    if close == 10 then
      done = true
    end
    rednet.send(0, close)
  end
end

function reset()
  close = 0
  done = true
  while done do
    turtle.back()
    close = close + 1
    if close == 20 then
      done = false
    end
  end
end

close = 0
turtle.refuel()
rednet.open("left")

while true do
  local id, message = rednet.receive()
  if message == "start" then
    race()
  end
  if message == "reset" then
    reset()
  end
end
```

#### Timer

The timer has functions as well and what it does is when the start function is called it will go through all of the code in the functionn rising a number by 0.1 every tenth of a second. The reset function will set the timer back to 0 and will clear the screen ready for the next race.

It listens for the **start** and **stop** messages from the main computer and starts or stops the timer. It listens for the **reset** message from the main computer to reset the timer and clear the monitor ready for a new race.

```
monitor = peripheral.wrap("back")
timer = 0
time = 0
racing = false
rednet.open("left")
monitor.setTextScale(5)
monitor.setTextColor(colors.red)
monitor.setBackgroundColor(colors.yellow)

function reset()
  timer = 0
  monitor.clear()
end
function race()
  racing = true
  while racing do
  local id, message = rednet.receive(0.1)
    timer = timer + 0.1
    monitor.setCursorPos(2, 3)
    monitor.write(timer)
    monitor.setCursorPos(5, 3)
    if timer >= 10 then
      monitor.setCursorPos(6, 3)
    end
    monitor.write("            ")
    if message == "stop" then
      time = timer
      rednet.send(22, time)
      racing = false
    end
  end
end
while true do
  local id, message = rednet.receive()
  if message == "start" then
    race()
  end
  if message == "reset" then
    reset()
  end
end
```

#### World record recorder

This code is for the world record. What it does is it will receive a message from the timer computer when the time has come through and it will do the < sign to check if the world record was broken and if it was it will set the world record to that time and then it will start displaying that time.

and displays a message if a new record is set. It listens for messages sent by the Timer computer

```
rednet.open("left")
monitor = peripheral.wrap("right")
wr = 9.9
monitor.setBackgroundColor(colors.yellow)
monitor.setTextColor(colors.red)
while true do
  monitor.setCursorPos(2, 3)
  monitor.setTextScale(5)
  monitor.write(wr)
  monitor.setCursorPos(5, 3)
  monitor.write("               ")
  local id, message = rednet.receive()
  if id == 19 then
    if message < wr then
      wr = message
      monitor.setTextScale(5)
      monitor.setCursorPos(2, 3)
      monitor.write("               ")
      monitor.setCursorPos(2, 3)
      monitor.write("NEW")
      sleep(1)
      monitor.setCursorPos(2, 3)
      monitor.write("                            ")
      monitor.setCursorPos(1, 3)
      monitor.write("WORLD")
      sleep(1)
      monitor.setCursorPos(1, 3)
      monitor.write("             ")
      monitor.setCursorPos(1, 3)
      monitor.write("RECORD")
      sleep(1)
      monitor.setCursorPos(1, 3)
      monitor.write("            ")
    end
  end
end
```