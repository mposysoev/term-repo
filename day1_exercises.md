# Day 1: the terminal, the cluster, and one plot

Someone ran four simulations of a box of water at 280, 300, 320 and 350 K.
One of them crashed. Today you download the four runs, find the broken one,
fix it, run it again on the cluster, bring the results back, and plot them.
That is the loop of the whole course, done once with small files.

### Rules for today

- Type the commands yourself.
- `Tab` completes a name. `↑` brings back the last command. `Ctrl+C` stops what is running. `q` leaves `less`.
- Lost? `pwd`, then `ls`. Most errors mean you are in the wrong folder.
- `USERNAME` below means your cluster username. Type your own name instead of the word.
- No spaces in file names. Use `_`.

---

## 1. Where am I?

Open a terminal: `Ctrl+Alt+T` on Ubuntu, the *Terminal* app on a Mac.

```bash
echo hello
pwd
ls
```

The text before the cursor is the *prompt*: `user@computer:folder$`. Read it before you type.

Make a folder for today and go into it:

```bash
cd
mkdir modelling_day1
cd modelling_day1
pwd
```

Move up and come back:

```bash
cd ..
pwd
cd modelling_day1
```

`cd` alone takes you home. `..` is the folder above. `~` is your home folder.

**Check:** `pwd` ends with `modelling_day1`.

---

## 2. Download the data

```bash
wget https://raw.githubusercontent.com/OWNER/REPO/main/terminal/tasks/water.tar.gz
ls
```

On a Mac, use `curl -LO` instead of `wget`, with the same address.

A `.tar.gz` file is a packed folder. Unpack it and look inside:

```bash
tar -xzf water.tar.gz
ls
cd water
ls
cat README.txt
ls T300
```

**Check:** `ls` in `water` shows `README.txt  T280  T300  T320  T350`.

---

## 3. Read the files

Still in `water`. A short file can be printed whole:

```bash
cat T300/md.mdp
```

A long file is better read in parts:

```bash
head -n 3 T300/conf.gro
tail -n 1 T300/conf.gro
```

**Q1.** How many atoms are in the box (second line of `conf.gro`)? How large is the box in nm (last line)?

Page through the log of one run:

```bash
less T300/md.log
```

Space goes down, `b` goes back, `/Temperature` then Enter searches, `q` quits.

Ask for the lines you want:

```bash
tail T300/md.log
grep Performance T300/md.log
```

**Q2.** How many nanoseconds per day did the 300 K run reach?

---

## 4. Which run crashed?

A run that ends well writes `Finished mdrun` at the end of `md.log`. Look in all four logs at once:

```bash
grep Finished */md.log
```

`*` means "any name", so `*/md.log` is the log in every folder. One folder is missing from the answer.
Read the end of its log:

```bash
tail -n 15 T350/md.log
```

Compare the settings of the four runs:

```bash
grep dt */md.mdp
```

**Q3.** Which run crashed, and why?

---

## 5. Fix the run

```bash
cd ~/modelling_day1/water/T350
ls
```

Keep the old files under new names: `cp` copies, `mv` renames.

```bash
cp md.mdp md_crashed.mdp
mv md.log md_crashed.log
ls
```

The two `.pdb` files are what the program saved as it crashed. Look at them, then delete them.
`rm` has no trash bin, so always `ls` the same name first:

```bash
ls step*
rm step*.pdb
ls
```

Now change the settings:

```bash
nano md.mdp
```

In `nano`, the arrow keys move (the mouse does not). `Ctrl+O`, then Enter, saves. `Ctrl+X` leaves.

Set `dt = 0.002`. The run should still be 500 ps long, so set `nsteps = 250000` (250000 × 0.002 ps = 500 ps). Save and leave.

```bash
grep dt md.mdp md_crashed.mdp
grep nsteps md.mdp
```

Try to start the run here:

```bash
bash run.sh
```

It refuses. Read why.

---

## 6. To the cluster and back

Open a second terminal. Keep the **left** one on your computer and the **right** one on the cluster.

**Right:** log in.

```bash
ssh USERNAME@mmkluster.fos.su.se
```

The first time, answer `yes`. The password does not appear while you type it. Type it and press Enter.
Look at the prompt: it now shows the cluster's name.

```bash
cd /nas_home/USERNAME
mkdir modelling_day1
ls
```

**Left:** send the folder. `scp` copies between computers, `-r` means a whole folder.
The colon `:` separates the computer from the path on it.

```bash
cd ~/modelling_day1/water
scp -r T350 USERNAME@mmkluster.fos.su.se:/nas_home/USERNAME/modelling_day1/
```

**Right:** run it on the cluster.

```bash
cd modelling_day1/T350
ls
bash run.sh
ls
tail md.log
```

**Left:** bring the folder back under a new name, so you can see that it came from the cluster.

```bash
scp -r USERNAME@mmkluster.fos.su.se:/nas_home/USERNAME/modelling_day1/T350 T350_cluster
ls
ls T350_cluster
```

**Right:** log out. The prompt shows your own computer again.

```bash
exit
```

**Check:** on the left, `ls T350_cluster` shows `temperature.xvg` and `density.xvg`.

`scp` always runs on your own computer, in both directions. `ssh` is only for working on the cluster.

---

## 7. One plot

Start Jupyter in `modelling_day1`, so that it can see the `water` folder:

```bash
cd ~/modelling_day1
jupyter notebook
```

A browser opens. **New → Python 3**. Click the name *Untitled* and call it `day1`.
This terminal is now busy; leave it open.

Type into the first cell, then press `Shift+Enter` to run it:

```python
print("hello from a code cell")
```

Next cell:

```python
import numpy as np
import matplotlib.pyplot as plt
```

Next cell. `.xvg` files start with lines beginning with `#` and `@`: `comments` tells `loadtxt` to skip them.

```python
data = np.loadtxt("water/T350_cluster/temperature.xvg", comments=["#", "@"])
time = data[:, 0]
temperature = data[:, 1]

plt.plot(time, temperature)
plt.xlabel("Time (ps)")
plt.ylabel("Temperature (K)")
plt.savefig("temperature_T350.png")
```

`data[:, 0]` means "every row, first column". Python counts from 0.

```python
print(temperature.mean())
```

**Q4.** Is the average close to the `ref-t` in `md.mdp`?

Now plot `density.xvg` from the same folder yourself: copy the plotting cell, change the file name,
the variable name and the axis label, and save `density_T350.png`.

**Q5.** The run started from a box made at 300 K. How many ps does the density need to settle?
What is the average density after that? (`density[100:].mean()` skips the first 100 frames.)

Check the pictures from another terminal:

```bash
cd ~/modelling_day1
ls
```

When you are done, go to the Jupyter terminal and press `Ctrl+C`.

---

## If you finish early

**A. Density against temperature.** Collect the average density of all four runs in one plot,
next to the measured density of water:

```python
temps = [280, 300, 320, 350]
folders = ["T280", "T300", "T320", "T350_cluster"]
mean_density = []
for folder in folders:
    d = np.loadtxt("water/" + folder + "/density.xvg", comments=["#", "@"])
    mean_density.append(d[100:, 1].mean())

plt.plot(temps, mean_density, "o-", label="SPC/E water, simulation")
plt.plot(temps, [999.9, 996.5, 989.4, 973.7], "s--", label="experiment")
plt.xlabel("Temperature (K)")
plt.ylabel("Density (kg/m$^3$)")
plt.legend()
plt.savefig("density_vs_T.png")
```

**B. A fifth temperature.** On the cluster, copy the fixed run, change the temperature, and run it:

```bash
cd /nas_home/USERNAME/modelling_day1
cp -r T350 T330
cd T330
nano md.mdp
bash run.sh
```

Set `ref-t = 330`. Bring `T330` back to `water/` with `scp -r` and add it to plot A.

**C. Look around.** `history` lists your commands. On the cluster, `mc` shows the folders in two panels (F10 quits).
