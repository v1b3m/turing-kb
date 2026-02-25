- Original site does not required the "Unit or apt #" before continuing
- The taskers are all generic for each task, we'll need to handle this so taskers for specific categories are the ones shown such as "Cleaning taskers"
- We could fake the calendar loader, currently shows calendar immediately
- Calendar font sizes, background colors, font colors etc do not match
- Error on hitting Calendar "Select & Continue"

```
# Unhandled Runtime Error

TypeError: Cannot read properties of undefined (reading 'map')

## Source

src/components/task/ConfrimFrequencyDetailsSection.tsx (297:18) @ CLEANING_FREQUENCY_SELECTION

  295 |               </Typography>
  296 |               <Box role="radiogroup">
> 297 |                 {CLEANING_FREQUENCY_SELECTION.map((option) => (
      |                  ^
  298 |                   <Box key={option.value}>
  299 |                     <Box
  300 |                       key={option.value}
```

