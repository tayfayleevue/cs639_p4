# P4 (6% of grade): Elasticsearch and 639 Madison Interactive Kibana Map

![Static Badge](https://img.shields.io/badge/Deadline-Oct%2028-blue)
![Static Badge](https://img.shields.io/badge/topics-elasticsearch%2C%20kibana-green)
![Static Badge](https://img.shields.io/badge/course-CS639-purple)

<details>
  <summary>If you love Friends, click me!</summary>
  <img src="assets/meme_p4.png">
</details>

### :outbox_tray: Github classroom link: [https://classroom.github.com/a/zHP2V1HZ](https://classroom.github.com/a/zHP2V1HZ)

![Interactive_Kibana_Map](assets/map.png)
<!-- **Github Classroom Invitation Link: [https://classroom.github.com/a/1lhNtVba](https://classroom.github.com/a/1lhNtVba)** -->

## Clarifications / fixes

* Local autograder (not the github actions one) accepts elastic version 8.15.3 as well now. 
* q2: "please load the entire document in the index"
* q3: Added hint
* q4: "filtering or searching within the `wiki` field"
* q5: Use a `match_phrase` query
* q7: var: `total_sum_arrests` changed to `total_arrests_sum`
* q9: Please `copy value of _source`

  
## :telescope: Overview

Learning Objectives:
* Indexing and Querying Data: Learn to index various data types and perform complex queries.
* Aggregation, Filtering and other data analysis queries: Learn to count places based on XYZ, filter them based on popularity/news mentions.
* Combining Data Sources: Integrate multiple sources of information (news, Wikipedia, Google Maps, Weather and custom scraping) to enrich the dataset.
* Geospatial Data Management: Understand how to handle and query GeoJSON data to represent geographic features.
* Data Visualization with Kibana: Use Kibana Maps to visualize the geospatial data dynamically and interactively.

You will be building a practical project while answering 10 cool Madison trivia questions!

--- 

## :hammer_and_wrench: Upgrade your VM to e2-medium 
- Stop and remove all running `docker` containers.
- Remove all extra/unnecessary data files. You can always redownload when you want to review those topics.
- We'll continue to have disk size as 25GB.
- Here's a helpful [tutorial](https://cloud.google.com/compute/docs/instances/changing-machine-type-of-stopped-instance) to upgrade your existing VM.
- **IMPORTANT**: It is very important that you go through this step first before you go through the installation for ElasticSearch. Otherwise, the installation might fail!

## :hammer_and_wrench: Setting up ElasticSearch using Docker

**Note** Developing in VS Code is way easier! (The extension: Remote-SSH is a life saver). Please consider switching to VS code instead of 10 floating terminals.

- In your VM, install Elastisearch and Kibana (with docker):
```bash
curl -fsSL https://elastic.co/start-local | sh
```

**This command will give you a password and an API key as shown below. Store both.**

![output_of_installing_elasticsearch_with_docker](assets/output_local_install.png)

- To restart your container, cd into `elastic-search-local`: 
```bash
docker stop es-local-dev
cd elastic-start-local
docker compose up -d
```

Sometimes you have to remove the elastic-start-local dir and start from scratch (container state becomes unhealthy when you re-start your VM sometimes)

- [Optional] Sanity check your elasticsearch connection. In your VM:
```bash
cd elastic-start-local
source .env
curl $ES_LOCAL_URL -H "Authorization: ApiKey ${ES_LOCAL_API_KEY}"
```

---


## :nut_and_bolt: Setting up `jupyter` and `python elasticsearch-client`

- Install `elasticsearch` so we can interact with elasticsearch via Python.

```bash
pip3 install elasticsearch
```

- Create a new directory (`p4`) to keep track of your p4 files and `cd` into it.
- Then, launch `jupyter` using the below command, and make note of the auto-generated URL.

```bash
jupyter notebook
```
(usually goes to port 8888)

OR if you want to specify a specific port:
```bash
jupyter notebook --port WXYZ
```

**😎 IT-Support Note:** If you see `jupyter command not found`, try this: 
```bash
export PATH=$PATH:~/.local/bin
```

- To access the `jupyter` session on your laptop, you must establish an `ssh` tunnel. Open another *local* terminal or powershell, then use either of the below commands to establish your tunnel.

```bash
ssh <USER>@IP -L localhost:WXYZ:localhost:WXYZ
```
or
```bash
gcloud compute ssh <VM_NAME> -- -L localhost:WXYZ:localhost:WXYZ
```

:point_right: Double-check that the second port matches the port in the URL.

- Now, open the jupyter URL in your browser, and create a new notebook file (File > New > Notebook). Make sure to save it as `p4.ipynb`.

---

## Good Practice: Spending time with your data first and deciding on the right tool!

Check out the JSONs, column names of csvs and skim the text data, at the very least!
This is how data in-the-wild exists. Scary right? 

Data Sources: 
* [Wikipedia API - PyPi](https://pypi.org/project/Wikipedia-API/)
* [Google Maps APIs](https://developers.google.com/maps/documentation/geocoding/overview). 
* [News API](https://newsapi.org/)

If you are curious about the data scraping part, see how I did it here: [data_scripts](./data_scripts) 

Scared more?

With unstructured data processing tools like Elasticsearch you can make sense of it and build your downstream application pretty fast :)

Since Elasticsearch literally creates an indexable/hash-searching data-structure within, querying/searching is extremely quick. 

#### The unstructured-ness, speed of searching and data-scalability is where simple libraries like Pandas (assuming you come up with your own data-structure first) lose.   

#### Here's a real app in deployment that switched to Elasticsearch for speed (5 days to 5 ms!): 

<details>
  <summary>PostGres vs Elasticsearch: For searching</summary>
  <img src="assets/postgresql_vs_elasticsearch.png">
</details>


---  

## :floppy_disk: Writing code in `p4.ipynb`  

1. Inside your notebook, use the appropriate `bash` command to download ```data.zip``` if your starting repo doesn't have it. Use this URL to download: https://github.com/CS639-Data-Management-for-Data-Science/f24/raw/915d4e52e18063d2217d513a5eee6a78be3e87cd/p4/data.zip
2. Write the appropriate `bash` command (in the same notebook) to unzip the data zip file.
3. **Make the connection to Elastisearch using the [python elasticsearch-client](https://www.elastic.co/guide/en/elasticsearch/client/python-api/current/index.html)** with your username (usually elastic) and password OR API key as demonstrated in the lecture. 

:warning: **Requirements (applies to all subsequent questions)**:

- Each question's resultant data dictionary must be stored into a [JSON file](https://docs.python.org/3/library/json.html) called `q<N>.json`, where `N` refers to the question number. Save all the files in a folder called `answers/`. For example, store q1's results using the below code:

```
response = client.whatever_rest_api_call(...)
with open('answers/q1.json', 'w') as f:
    json.dump(dict(response), f, indent=4)
```

- Make sure to go back to the cell containing import statements and include `import json` (and other necessary libraries, as needed).

## :blue_book: Section 1: Loading Different Types of Raw Data (2 questions)
> Objectives: To learn dataloading & parsing into Elastic Common Schema (ECS) using Elasticsearch bulk upload (dynamic mapping). Then realise that the unstructured data structure offered by Elasticsearch allows updating an existing schema and also allows explicit mapping.

#### Q1: Save the basic information about your cluster in q1.json. [0.3 points]

#### Q2: Create an index `madmap` and bulk load all JSONs (the entire document) except `places.json` [0.5 points]

* Save the dynamic mapping generated by elasticsearch as q2.json
  * Consider writing a function to save the mapping, as you'll be doing the same thing with Q3

#### Q3 Update the generated mapping with the index-able field: `wiki` (type: text) and load all the text files. [0.5 points]

* Save the dynamic mapping generated by elasticsearch as q3.json

<details>
  <summary>
    Hint:
  </summary>
  <p>Use 'put_mapping' to update the schema first, then load the text files</p>
</details>


---


## 💎 Section 2: Madison and UW-M Trivia! (3 questions)

> Objectives: To learn query with Elasticsearch


#### Q4: What are the 3 biggest football rivalries of Wisconsin Badgers? [0.5 points]

* Write a `simple_query_string` query, filtering or searching within the `wiki` field and save the response as q4.json


**NOTE:** You will pass the autograder if your search `_score` is higher than 6   

<details>
  <summary>
    Hint:
  </summary>
  <p>Read up on Boosting (https://weng.gitbooks.io/elasticsearch-in-action/content/chapter6_searching_with_relevancy/63boosting.html) and 'Use Cases of Boosting' (https://stackoverflow.com/a/73957996)</p>
</details>


#### Q5: Hmmm that is still a lot of text to look through! Can you highlight what you found in Q4? [0.5 points]

* Highlight the `wiki` field.
* Use a `match_phrase` query
* Save `response['hits']['hits'][0]['highlight']` as q5.json


#### Q6: Do you know when and where Coldplay is coming to town? :eyes: Write a match_phrase search query. [0.5 points]

A. Write a source filter on `title`, `geoLocation`, and `WhenIsItHappening` to make the search efficient. 
B. Highlight the title of the match. 
C. Save that response as q6.json.
D. Book your tickets early :)

* Remember this is a search engine so you might see a lot of source hits!

#### Q7: How many people were arrested in the State Street Halloween Party from from 2001 to 2019? [0.8 points]

**NOTE:**  Please name your sum variable `total_arrests_sum` Save the `response['aggregations']` to q7.json.

<details>
  <summary>
    Caution!
  </summary>
  If you load your data multiple times, Elasticsearch appends the documents! You might see a higher count due to duplication.
</details>

---

## 🖌️ Section 3: Interactive Visualization: Making the 639 Madison maps application

> Objectives: Learn interactive geospatial visualization with Kibana and have fun 

#### Q8: Load places.json into Kibana Maps. Try adding filters to categorize based on `place_types` [0.6 points]

* These categories go together but you get points regardless (as long as you load, display and submit the png :) )

1. `bars`, `cafes` and `restaurants`
2. `park`, `gyms` and `water_sports`
3. `political`, `museums` and `tourism`
4. `gas` and `groceries`
5. `clubs`
6. `uw` 

**NOTE:** In Import Data (Advanced), don't forget to change the mapping to `geo_point` and name field to `keyword`

* Fit to data view -> Fullscreen and then take a screenshot of your screen. Name the file q8.png and add it to your repository.


#### Q9: Why is this any better than Google Maps? From the interactive viewer, filter the most expensive place and find the corresponding document in Kibana. [0.5 points]

* Copy value of _source from Kibana
* Save that as q9.json

<details>
  <summary>If you are feeling lost in the interactive viewer:</summary>
  Analytics --> Discover --> Your filter query in the top bar --> Hover over your document(s) --> Copy value of _source --> Paste in a file named q9.json
</details>

#### Q10: Find the nearest cafe from Computer Sciences Dept in this dataset using the `draw distance` filter tool. Set the cafe marker color green and computer sciences to red. [0.6 points]

* Submit a screenshot of the region filter as q10.png

---

<details>
  <summary>Open when you are done! (Trust me it's better that way)</summary>
  <img src="assets/michael_proud.png">
</details>

---

## :outbox_tray: Submission

- GitHub Classroom will automatically select your last pushed commit before the deadline as your submission.
- You can also run `autograder.py` locally by invoking `pytest autograder.py`. If you don't have pytest: `pip3 install pytest`

- The structure of the required files for your submissions is as follows:
```
p4-<your_team_name>
|--- README.md (list names of team members at the top)
|--- p4.ipynb
|--- answers/
    ├── q1.json
    ├── q2.json
    ├── q3.json
    ├── q4.json
    ├── q5.json
    ├── q6.json
    ├── q7.json
    ├── q8.png
    ├── q9.json
    └── q10.png
```


## :trophy: Testing
- GitHub Classroom has a simple autograder which will (auto-)execute each time you _push_ new commits. You can also manually re-run it using the GitHub UI.

- Just because the autograder passes doesn't mean you'll get full points — we will still review your notebook manually.
