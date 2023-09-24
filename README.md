# Covid-Portfolio-Project
<img width="599" alt="Covid Project" src="https://user-images.githubusercontent.com/108308205/177436450-e23dd47b-525b-4bf0-9081-d8a1b63ec82b.png">

This project is designed to represent my Technical Skills in SQL for Data Exploration. I will then visualize the data by using Tableau for further analysis. 

## Prepare
- The Data Source for this Project is [Our World in Data](https://ourworldindata.org/covid-deaths). 
- To start my Project, I need to make sure that my Dataset is unbiased and credible. As it follows, this Dataset is reliable, original, and comprehensive. Furthermore, the Dataset is current since the data on the coronavirus pandemic is updated daily on [Our World in Data](https://ourworldindata.org/covid-deaths).

## Process
- I broke down my Dataset into two separate tables (*CovidDeaths* and *CovidVaccinations*) by using Microsoft Excel.

After uploading the cleaned Datasets, the following were done to ensure that the data is clean before proceeding to the *Data Exploration Phase*.

```sql
-- View and Examine Tables

-- 85,171 records from CovidDeaths table
SELECT *
FROM `expanded-bebop-352104.covidproject.CovidDeaths`

-- 85,171 records from CovidVaccinations table
SELECT *
FROM `expanded-bebop-352104.covidproject.CovidVaccinations`

```

## Data Exploration

```sql 
-- Select Data that I am going to be starting with 

SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM `expanded-bebop-352104.covidproject.CovidDeaths`
WHERE continent is not null
ORDER BY 1,2

-- Total Cases vs Total Deaths
-- Shows likelihood of dying if you contract covid in United States

SELECT Location, date, total_cases, total_deaths, (total_cases/total_deaths)*100 AS DeathPercentage
FROM `expanded-bebop-352104.covidproject.CovidDeaths`
WHERE Location = 'United States'
  AND continent is not null
ORDER BY 1,2

-- Total Cases vs Population
-- Shows what percentage of population infected with Covid in United States

SELECT Location, date, total_cases, population, (total_cases/population)*100 AS PopulationInfectedPercentage
FROM `expanded-bebop-352104.covidproject.CovidDeaths`
WHERE Location = 'United States'
ORDER BY 1,2


-- Conutries with Highest Infection Rate compared to Population

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From `expanded-bebop-352104.covidproject.CovidDeaths`
--Where location like '%states%'
Group by Location, Population
order by PercentPopulationInfected desc

-- Countries with Highest Death Count per Population

Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From `expanded-bebop-352104.covidproject.CovidDeaths`
--Where location like '%states%'
Where continent is not null 
Group by Location
order by TotalDeathCount desc



-- BREAKING THINGS DOWN BY CONTINENT

-- Showing contintents with the highest death count per population

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From `expanded-bebop-352104.covidproject.CovidDeaths`
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc



-- GLOBAL NUMBERS

Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From `expanded-bebop-352104.covidproject.CovidDeaths`
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2



-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as int)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From `expanded-bebop-352104.covidproject.CovidDeaths` dea
Join `expanded-bebop-352104.covidproject.CovidVaccinations` vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3


-- Using CTE to perform Calculation on Partition By in previous query

With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as int)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From `expanded-bebop-352104.covidproject.CovidDeaths` dea
Join `expanded-bebop-352104.covidproject.CovidVaccinations` vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac



-- Using Temp Table to perform Calculation on Partition By in previous query

DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From `expanded-bebop-352104.covidproject.CovidDeaths` dea
Join `expanded-bebop-352104.covidproject.CovidVaccinations` vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated




-- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as int)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From `expanded-bebop-352104.covidproject.CovidDeaths` dea
Join `expanded-bebop-352104.covidproject.CovidVaccinations` vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null

```

## Share
In this phase of Analysis, Tableau Public was used to visualize the analysis made on this study.

![Covid](https://user-images.githubusercontent.com/108308205/177652333-a351ca59-2b94-4b34-9c4b-aae86f314773.png)
