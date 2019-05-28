# Frontend

## Search process happy path

1. Get text from user (e.g., "brooklyn")
2. Use [Google Place Search API](https://developers.google.com/places/web-service/search) to determine what place they mean (get place ID)
3. Use [Google Place Details API](https://developers.google.com/places/web-service/details) to find full place details
    - Need state & either city or county
4. Check city & state against PSAP list
    - `A`/`M`/`NC` - Good
    - `O` - Bad
    - `S` - Good, handled by primary PSAP (primary can sometimes be found in comments)
    - Not found - Bad

### Unhappy cases

- No results from user text
- Multiple results from user text
- One result, but it's the wrong one
- Place not in US

# Backend

The backend consists of two parts, which communicate through a GitHub repo.

## Intake

A simple cronjob that runs monthly(?). The purpose of this is to create a paper trail of the FCC 

1. Check the current file version from the FCC.
    - If it's a version we've already processed, stop here.
2. Download the CSV.
3. Save the CSV to the database repo.
4. Overwrite `current.csv` with the new CSV.
    - This could be accomplished through a file pointer, but overwriting a persistent file gives an easy-to-use diff for each update.
4. Commit the changes to the database repo.

## API

### The actual API

`/api/v0.1/status`

Requires the following query parameters:

-   `state` or `territory` (interchangeable) - The two-letter ANSI INCITS 38:2009 code (which is also the USPS postal code, plus `UM` for the Minor Outlying Islands which do not have a USPS code)
-   One of the following:
    -   `city`
    -   `county` (Note that this does _not_ say "country")

~~City & county are not case– or diacritic–sensitive, but **are** whitespace-sensitive. (For example, `San Jose` , `san josé`, and `san jose` will all work for San José, CA, but `SanJosé` will not.)~~
This is a future improvement, since SQL does not offer these as built-in options. For now, city & county must exactly match what are in the FCC CSV. This hopefully won't be a problem if cleansing user input through Google.

### DB Schema

| Name             | Type                                 | Required? | Notes                     |
|------------------|--------------------------------------|-----------|---------------------------|
| ID Number        | Int                                  | Required  |                           |
| City             | String                               | Optional  |                           |
| County           | String                               | Required  |                           |
| State            | String                               | Required  | Could also be a territory |
| PSAP Name        | String                               | Required  |                           |
| Status           | One of: primary, secondary, orphaned | Required  |                           |
| Last update date | Date                                 | Required  |                           |

### Getting new data

