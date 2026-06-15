There's a possible bug with deductions:

1. Say I add a property tax of 1000
2. Then I add another still under property for 200
3. The total in salt becomes 1200, this is an addition (appears correct)
4. However, you then notice the value was not initially zero, so why was the original value totally replaced with 1000, and not added.
5. Confirm this is the expected behavior and not a bug
6. I'd assume other deductions too are affected by similar logic, so in case this is a bug, we need to check all deductions

![[Pasted image 20260616001037.png]]

