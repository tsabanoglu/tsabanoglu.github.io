---
title: Adventures with Claude Code
date: 2026-02-08
published: true
---

#### Building things that build things
A few weeks ago I installed the Claude Code cli tool and my programming habits changed overnight. I never cared for IDE-based AI assistants or autocompletion, and solved most of my data and programming-related questions by asking an LLM, making an effort to understand what was going on in the suggested code before I copy-pasted the solution. Maybe because my first introduction to programming was via another cli-based program (R's Swirl package), or maybe because Claude Code's spinner verbs were just the tonic to my whimsical mind, I felt immediately at home and got on building things, starting with a project I've been harbouring for a while.

I like making things. More specifically, I like hand knitting garments for myself. And although I'm not a software engineer who writes programs, I'm familiar with a programming language (`Python`) enough to build little tools for myself. This time, though, I wanted to see if I can go the whole length and build an entire package that helps knitters create a sweater pattern based on minimal gauge and size input. And although I'd started `sweater` before my introduction to Claude Code, I rewrote and resolved so many thorny parts with it. In this post I want to talk a bit about the project along with the main hurdles and learnings. 


#### Gauge in, pattern out

At the highest level, `sweater` is designed to give you a full pattern for a top-down raglan sweater based on a few input parameters - gauge info, garment size and neckband ribbing - a user provides to the Sweater class. The `Sweater` class acts as a facade and orchestrates all the components needed to build a pattern. Below is an example output which uses the `render_text()` method to generate a full sweater pattern.

```python
my_sweater = Sweater(sts_per_10_cm=22, rows_per_10_cm=14, size="M", rib_pref=1)
print(my_sweater.render_text())
```

Whether you're following a pattern, or just vibe knitting (something I do very often), it is considered best practice to make a 10 cm x 10 cm sample called a "gauge swatch" first to make sure your finished object will end up being the correct size. Since needle size, yarn weight and knitting tension will ultimately determine how big or small a garment turns out, a swatch made with your own knitting specs is going to save you a lot of headache. The number of stitches and rows per 10 cm is therefore the building block of this tool as well. Taking into account universal measurements through sizes XS to XL, the gauge kicks off the building process that starts with calculating the number of stitches to cast on.


#### Dataclasses as data holders
Knitting involves a lot of arithmetic, but also a lot of data structuring as well since we need the values that result from those computations and pass them between various sections of a pattern. Dataclasses were new to me before this project, and I'm thankful I got to find out about them as they turned out to be perfect for modelling the structure of a sweater. The `@dataclass` decorator automatically generates methods like `__init__`, `__repr__`, `__eq__` and all the other boilerplate code that you're expected to write in a regular class. This means less code to write and read through. 

Dataclasses also require type annotations for each field, a huge readability win for me and hopefully for whoever uses this tool. When a single class mixes strings (size names), integers (stitch counts), floats (measurements), and booleans (construction choices), those annotations make it immediately clear what each field expects to anyone using these classes.


```python
@dataclass(frozen=True)
class SeparatorPlan:
    size: str
    body_sts_end_yoke: int
    sleeve_sts_end_yoke: int
    underarm_each: int = 8
    rib_multiple: int = 2
    include_raglan_lines_in_body: bool = False
    separate_at_row: Optional[int] = None
```
#### Standalone modularity

Between casting on for the neck band and binding off the hem, there are several steps and components to constructing a sweater that require a lot of calculation and modification. The package naturally treats these as part of the main `Sweater.build()` pipeline, but some of these components can also be as used as standalone tools. For instance, you can use the `SleevePlan` independently to design sleeve shaping and calculate a decrease schedule for the sleeves sections of a garment. 


```python
standalone_sleeves = SleevePlan(
    size = "M",
    start_sts = 80,
    length_rows = 100, # the desired length of your sleeves in terms of no. of rows (rather than cm)
    gauge_rows_per_cm = 2.8, # this is the key derived data
    target_cuff_sts = 52, # desired width of cuffs
    rib_multiple = 4, # 2x2 ribbing
)

schedule = standalone_sleeves.schedule_rows() # schedule_rows() method
dec_rows = [i+1 for i, is_dec in enumerate(schedule) if is_dec]
print(f"Decrease on rows: {dec_rows}")

```

You can take advantage of the `SleevePlan` class in two modes. 

In Target Mode, user specifies the number of stitches they want to end with. 

```python
target_sleeve = SleevePlan(
    size = "M", start_sts = 80, length_rows = 100, gauge_rows_per_cm = 2.8,
    target_cuff_sts = 52, # this parameter triggers the target mode
)
```

In Fixed Mode, user specifies the frequency of decrease rows. For example, you want to decrease every 8 rows. 

```python
fixed_sleeve = SleevePlan(
    size = "M", start_sts = 80, length_rows = 100, gauge_rows_per_cm = 2.8,
    decrease_every_rows = 8, # this parameter triggers the fixed mode
)
```


#### Jumping at the deep end: German short rows for neck shaping

By default, `sweater` creates a simple raglan pattern with no back shaping, but the class also has the optional `short_rows` parameter, which, when enabled, triggers `ShortRowPlan` to create back shaping for your sweater using the German Short Rows method. How many rounds of short rows are needed to be worked is determined by the extra depth you provide as input. 

```python
sweater = Sweater(
    sts_per_10cm = 22,
    rows_per_10cm = 14,
    size = "M",
    use_short_rows = True,      # enables short rows shaping
    short_row_depth_cm = 5.0    # how much depth - determines how many rounds of short rows
)
```

Short row shaping is not a must in sweater construction, but it makes for a better fit. Most modern patterns for sweaters, jackets and blouses incorporate them, and once you get the hang of it, it will make you feel much more confident about your knitting skills. Short rows help raise the neckline of a sweater by adding extra rows to the work vertically, so that the back section of the sweater is higher than the front. Below is a rough sketch of a sweater where you can see this lifting effect.

![Raised neckline diagram](/assets/images/neckline-diagram.jpg)


The short row shaping logic was probably the trickiest part of this project where Claude Code was the most helpful. Working short rows themselves doesn't add new stitches to your work, they add new rows and create vertical height. So, essentially you're working towards part of the next row. However, in a raglan sweater the short row rounds include working the raglan increases at the back raglan lines. Therefore while the short rows are adding height to the back, the raglan increases are simultaneously adding stitches to both the back and the shoulders. This in turn meant that cast-on and initial split had to follow a different logic if short row parameter is enabled to make sure that at the end of short row rounds, the front and back sections have the same number of stitches. Representing this mechanical process mathematically and programmatically without breaking everything else had a steep learning curve, and like many other things in life, picking up a pen and paper to represent the process step by step helped a ton before I attempted to program it. 

Below is another rough sketch where 6 rounds of short rows are worked on the back section and back-shoulders (3 rows on the right side of the work, 3 rows on the wrong side). Supposing we start off with 11 stitches at the back section, having worked short rows on the back section and shoulders, we get 12 new stitches; 6 of which having been added to the back section (and the 6 other to the shoulder sections). By the end of it, the total stitch count is 17, the same as the front section which we didn't touch. 

![Short rows diagram](/assets/images/short-rows-diagram.jpg)

The short rows construction was an interesting exercise also because I had to juggle a few other constraints. Since this sweater only uses neckline shaping for the back section, the number of short row rounds is bounded by the initial shoulder stitch count. This means the short rows schedule shouldn't extend past the raglan lines into the front. This is where things got properly mathematical. And once I managed to crack the right algorithm, it became a very satisfying constraint satisfaction problem to solve: finding a valid short row configuration that fits within the geometric limits of the sweater.



As with `SleevePlan`, `ShortRowPlan` can also be used as a standalone component. 


```python
short_rows = ShortRowPlan(
    size="M",
    gauge_rows_per_cm=2.8,
    back_sts=22,
    right_shoulder_sts=16,
    left_shoulder_sts=16,
    short_row_depth_cm=5.0,
)

```


#### Next steps: test a sweater!
I'm pretty happy about how my raglan sweater pattern generator has turned out based on the tests and demos I ran. I'm, however, aware that the real stress test is knitting an actual sweater based on the generated pattern. #1 item on my TODO for this project is to start knitting one in February and going back to the package and this post. 


