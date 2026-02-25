- task-rabbit-iota.web.app

### Hire page concerns

- Date field should consider the availability of that particular tasker
- Look into the recommendations page and consider how they calculate the availability of the tasker, we should then replicate this logic
- We should persist the details when returning back to the hire page 
- There's a template on the details page, we should make sure every other functionality is maintained on the hire page.
	- Reach out to Jagadeesh about the changes he's going to implement on the details page and ensure we have the same functionality on the hire page save for the header and the date and time picker

### Data Quality

- For most of the data, if I am choosing a particular location, we lack taskers in multiple locations
- We want to have equal distribution of taskers. 
- The work locations of the taskers in the UI are mapped based on `src/data/locations/locations.json`. This means we need to update the `src/data/all-taskers.json` to use the locations from `locations.json` instead.
- check how `api/locations` works
- workLocations has 164 locations
- according to the locations, there's only 139 locations
- 67 locations in workLocations which are not in the locations api (data)
- `taskers` data should be spread equally across the 139 locations (`locations.json`)
- `taskers` data should be spread equally across the service Id's
	- say 2251 has 5 taskers, any other serviceId should have a number 5 with a little error margin say +-2ß

- recommendations page shows available taskers

- whatever `serviceId` and `location` I choose, I need to see at least 5 taskers
	- We need to first do the maths for this

- if we need 10000 taskers, then we can split the files into `tasker-n.json` with `n < 5`
- The API can then merge these files and provide these taskers into a single entity

---

### OTS

- We're going to follow the OTS (Off The Shelf) gyms
- Standardizations and guidelines for this to make the gym OTS ready
	- They will be sharing the required document soon
	- We may need to implement the prompts and verifiers soon ~30 prompts
	- 