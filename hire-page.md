We are going to create a hire page, this page will be opened when a user navigates using "Select & Continue" from a tasker's profile page.

Unlike the normal booking flow with 4 steps, this page will only have two steps. So the stepper should also reflect this with as minimal changes as possible. We need to plan this carefully. 

1. Determine if it's possible to reuse the stepper but with configuration that determines how many steps it shows.
	1. Are there better approaches than my suggestions.

## The hire page

### How we navigate there

1. Go home: `/`
2. Scroll down to recommended taskers
3. Click "View Tasker profile"
4. The taskers skills will be shown with "Select & Continue" buttons
5. Select any of the available skills
6. The next page should be the hire page we're introducing
	1. Path: /book/{book-id}/hire
7. Check how other pages determine the book service Id (book-id) and reuse logic
	1. From the services page `/services`
	2. You could select a service such as `/services/handyman/electrical-work`
	3. Then select "Book Now"
	4. This will take you to `book/2064/details`
	5. The 2064 is the `book-id`
8. The book details page contains most of the components we need for the hire page with one new component
	1. Since we already know the tasker we're booking — hence no recommendations page needed, the hire page's second component will contain two dropdown selectors for the schedule. This means from the hire page, we go straight to confirmation
	2. The dropdowns use normal select html elements
		1. However, they are styled to match the site's theme

## Screenshots

![[Pasted image 20260206150324.png]]

![[Pasted image 20260206150339.png]]

![[Pasted image 20260206150349.png]]

![[Pasted image 20260206150421.png]]

From this page, we go direct to confirm

---

### 002

On the recommendations card where we navigate to the Tasker's profile, there are skill buttons such as "Shower Repair for $72/hr" etc. Clicking any of these buttons should skip the profile page all together and go straight to the hire page.

Ensure no data is lost in this navigation pathway.

----

### 003

For the hire page, the last item should still show continue, not "Review and Continue". We had this but I believe the merge overwrote it. So go through the git history to confirm and add this back.
