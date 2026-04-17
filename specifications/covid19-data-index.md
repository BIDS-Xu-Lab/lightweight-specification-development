I want to create a website for covid19dataindex.org. 

# Tech stack

- This website is based on Vue.js and Vite. 
- The source code is in the [web](web) directory.
- The website can be deployed on github pages without any server side.
- The search function is served by client side without server side, you can use itemjs to implement the search function with other faceted search.
- The website UI is based on PrimeVue and TailwindCSS.
- Icons will use fontawesome and primeicons.
- Visualization will use ECharts and also Vue-ECharts.
- The website will use Pinia for state management.
- Please install the dependencies and run the website locally before you start to write the code.

# Data

I want to index the following data which will be provided by a tsv file `covid19-data-index.tsv` in the public directory. Each row in the tsv file is a data record.

The data record is about the COVID-19 dataset. It includes the following fields:

- `id`: The unique identifier for the data record.
- `title`: The title of the data record.
- `description`: The description of the data record.
- `organization`: The organization of the data record.
- `data_type`: The type of the data record. such as Epidemiology, 
- `data_urls`: The URLs of the data record.
- `source_url`: The source URL of the data record.
- `authors`: The authors of the data record.
- `source`: The source of the data record.
- `country`: The country of the data record.
- `date_updated`: The last updated date of the data record.
- `date_created`: The date created of the data record.

# Pages

This website contains several pages:

- `/`: The home page, showing statistics of the data records, links to other pages, popular data records, etc.
- `/search`: The search page, showing the search results of the data records.
- `/about`: The about page, showing the about information of the website.
- `/submit`: The submit page, showing the form to submit a new data record. 
- `/detail/:id`: The data record page, showing the detail information of the data record in a table.
- `/feedback`: The feedback page, showing the form to submit feedback to the website.
Please create a prompt for the composer to generate the code for the website.

