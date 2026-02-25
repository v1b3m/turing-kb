We are going to do an analysis of the current state of the project.
1. What was implemented?
	1. Is it partial implementation?
	2. Is it complete?
	3. Does it share logic and/or components with other branches?

### These are the task categories we are dealing with:

|   |
|---|
|Task Categories|
|IKEA Furniture Assembly|
|Moving|
|Truck Assisted Help Moving|
|Furniture Assembly|
|General Mounting|
|Cleaning|
|TV Mounting|
|Heavy Lifting & Loading|
|Electrical Work|
|Plumbing help|
|Yard Work|
|Trash & Furniture Removal|
|Indoor Painting|
|Furniture Repair|
|Run Errands|
|Landscaping|
|Flooring|
|Wall Repair|

### Analysis so far

#### Cleaning

- Implemented in branch `fb-cleaning`
	- PR up at https://github.com/turing-rlgym/task-rabbit/pull/11
- Original site does not required the "Unit or apt #" before continuing
- The taskers are all generic(the same) for each task(category), we'll need to handle this so taskers for specific categories are the ones shown such as "Cleaning taskers"
- We could fake the calendar loader, currently shows calendar immediately
- Calendar font sizes, background colors, font colors etc do not match
- Error on hitting Calendar "Select & Continue"

#### General Mounting

- No implementation yet

#### Furniture Assembly

- Implemented in `fb-furniture-assembly`
	- PR https://github.com/turing-rlgym/task-rabbit/pull/9
- UI matches TaskRabbit with some quirks

#### Heavy Lifting & Loading

There's a page on the main branch with a very minimal implementation at `/heavy-lifting`

### Action Items

1. Do an analysis of all the features
2. Check all branches, pull remote branches if necessary
3. Write analysis to `/Users/v1b3m/Dev/KnowledgeBase/turing-kb/features.md`
4. 