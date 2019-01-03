# UCFParking-API
This is a 2-in-1 project. An unofficial API wrapper for [UCF's parking service](http://secure.parking.ucf.edu/GarageCount/) hosted as a Python app on Heroku, and a WIP website to view a graph of the data. Why did I make this you ask? Instead of making a request to UCF's parking website and scraping the HTML, it now becomes as easy as making a request to [this API](https://ucf-garages.herokuapp.com/api) and parsing the json.

How exactly is it 2-in-1 you ask? Well, making a request to the `/api` route returns a JSON response with info about each parking garage (spaces taken, percent full, etc). Making a request to the `/data/all` route returns a JSON response with info about how full each garage was from January to the current date (<b>BEWARE</b>, this will return a lot of JSON as each hour passes). The list is updated at the top of every hour every day. To view a specific date, make a request to `/data/month/{month}/day/{day}` where `{month}` is an int representing the month (1 for January, 2 for February, etc) and `{day}` is an int representing the number day of that month. For example, [`https://ucf-garages.herokuapp.com/data/month/1/day/2`](https://ucf-garages.herokuapp.com/data/month/1/day/2) returns how full each garage was on January 2nd. Any date in the future will just return an empty JSON array that looks like this: `{ "data": [] }`

# How does it work?
Heroku scheduler is a Heroku addon that can run a command at set intervals. Every hour, Heroku runs the `curl` command to the `/add` route (which requires a key) which scrapes UCF's parking site, extracts the garage info, and saves it to a PostgreSQL database. The `/add` route requires a key to prevent a regular user from making a request and adding data outside of that hourly interval. The table looks something like this (the values aren't exact):

Date                       |id |garage_data                                             |month |day
---------------------------|---|--------------------------------------------------------|------|----
2019-01-02T22:00:49.044984 |10  |{"garages": [{"name": "Garage A", "max_spaces": 1623...]|1     |2
2019-01-02T23:01:23.357748 |11  |{"garages": [{"name": "Garage A", "max_spaces": 1623...]|1     |2
2019-01-02T00:00:45.357748 |12  |{"garages": [{"name": "Garage A", "max_spaces": 1623...]|1     |3

### Sidenote
`config.py` contains 2 import things: The url of the PostgreSQL database and the key need to acces the `/add` route. If you want to build this yourself, you'll need to set up the database and generate a random key, something like `c52452a7-4f68-4033-a40b-31ec188e5c30` (if you want to prevent access the the `/add` route). Once you've done that, create a `config.py` file that looks like this:
```python
DATABASE_URL = 'postgres://some-awesome-url-here'
# This will be some random key you'll generate.
# Don't use this. I'd recommend using uuid4() from the uuid lib. 
KEY = 'c52452a7-4f68-4033-a40b-31ec188e5c30'
```

### Example request to "/api"
Using Python 3.x
```python
>>> from requests import get
>>> from json import dumps
>>> res = get('https://ucf-garages.herokuapp.com/api')
>>> dumps(res.json(), indent=3)
{
   "garages": [
      {
         "max_spaces": 1623,
         "name": "Garage A",
         "percent_full": 0.0,
         "spaces_filled": 0,
         "spaces_left": 1623
      },
      {
         "max_spaces": 1259,
         "name": "Garage B",
         "percent_full": 38.6,
         "spaces_filled": 486,
         "spaces_left": 773
      },
      {
         "max_spaces": 1852,
         "name": "Garage C",
         "percent_full": 0.0,
         "spaces_filled": 0,
         "spaces_left": 1852
      },
      {
         "max_spaces": 1241,
         "name": "Garage D",
         "percent_full": 0.0,
         "spaces_filled": 0,
         "spaces_left": 1241
      },
      {
         "max_spaces": 1284,
         "name": "Garage H",
         "percent_full": 0.0,
         "spaces_filled": 0,
         "spaces_left": 1284
      },
      {
         "max_spaces": 1231,
         "name": "Garage I",
         "percent_full": 0.0,
         "spaces_filled": 0,
         "spaces_left": 1231
      },
      {
         "max_spaces": 1007,
         "name": "Garage Libra",
         "percent_full": 11.32,
         "spaces_filled": 114,
         "spaces_left": 893
      }
   ]
}
```

### Example request to "/data/month/1/day/2"
Using Python 3.x
```python
>>> from requests import get
>>> from json import dumps
>>> res = get('https://ucf-garages.herokuapp.com/data/month/1/day/2')
>>> dumps(res.json(), indent=3)
{
   "data": [
   {
      "date": "2019-01-02T03:00:49.044984",
      "month": 1,
      "day": 2,
      "id": 4,
      "garage_data": {
         "garages": [
            {
               "max_spaces": 1623,
               "name": "Garage A",
               "percent_full": 0.0,
               "spaces_filled": 0,
               "spaces_left": 1623
            },
            {
               "max_spaces": 1259,
               "name": "Garage B",
               "percent_full": 51.31,
               "spaces_filled": 646,
               "spaces_left": 613
            },
            {
               ...
               # Lots of JSON goodnes below, you
               # get the idea. 'garage_data' just
               # contains, well, garage data (See "/api" above).
               ...
            }
           ]
         }
      }
   ]
}
```
