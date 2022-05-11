# Covid_SQL_Project
Tasks made it: n\
1- Download the latest covid data from https://ourworldindata.org in a csv format.n\
2- Create 2 excel workbooks: "Deaths" & "Vaccinations". n\
3- Use of the following querys to retrieve the data: n\

select *
from PortfolioProject..covid_data_deaths
where continent is not null
order by 3,4

--select *
--from PortfolioProject..covid_data_vaccinations
--order by 3,4
--where continent is not null

--SELECTING DATA THAT I GOING TO USE:

select location, date, total_cases, new_cases, total_deaths, population
from PortfolioProject..covid_data_deaths
where continent is not null
order by 1, 2

-- Looking for total_cases VS total_deaths
select location, date, total_cases, total_deaths, round((total_deaths/total_cases)*100, 2) as death_percentage
from PortfolioProject..covid_data_deaths
where continent is not null
order by 1, 2

-- United State Death Rate:
select location, date, total_cases, total_deaths, round((total_deaths/total_cases)*100, 2) as death_percentage
from PortfolioProject..covid_data_deaths
where location like '%%states%%' and continent is not null
order by 1, 2

-- Total_cases VS population (Infection Rate)
select location, date, population, total_cases, round((total_cases/population)*100, 2) as infection_rate
from PortfolioProject..covid_data_deaths
where continent is not null
order by infection_rate desc

-- Total_cases VS population (Infection Rate) in the United States
select location, date, population, total_cases, round((total_cases/population)*100, 2) as infection_rate
from PortfolioProject..covid_data_deaths
where location like '%%states%%' and continent is not null
order by infection_rate desc

-- Countries with higher infection_rate compared with population (Argentina en el puesto 17)

select location, population, max(total_cases) as highest_infection_count, max(round((total_cases/population)*100, 2)) as percent_population_infected
from PortfolioProject..covid_data_deaths
where continent is not null
group by location, population
order by percent_population_infected desc

-- Countries with highest death count

select location, max(cast(total_deaths as int)) as death_count
from PortfolioProject..covid_data_deaths
where continent is not null
group by location
order by death_count desc

-- Continents with highest death count

select continent, max(cast(total_deaths as int)) as death_count
from PortfolioProject..covid_data_deaths
where continent is not null
group by continent
order by death_count desc

-- Global Death Percentage

select sum(new_cases) as total_new_cases, sum(cast(new_deaths as int)) as total_new_deaths, round(sum(cast
(new_deaths as int))/sum(new_cases)*100, 3) as death_percentage
from PortfolioProject..covid_data_deaths
where continent is not null
order by 1, 2

-- Total Population VS Vaccinations

select deaths.continent, deaths.location, deaths.date, deaths.population, vaccinations.new_vaccinations,
sum(convert(int, vaccinations.new_vaccinations)) OVER (Partition by deaths.location Order By deaths.location,
deaths.Date) as rolling_count_people_vaccinated
from PortfolioProject..covid_data_deaths deaths
join PortfolioProject..covid_data_vaccinations vaccinations
	on deaths.location = vaccinations.location
	and deaths.date = vaccinations.date
where deaths.continent is not null
order by 1,2

-- CTE 

with people_vs_vacciantions (continent, location, date, population, new_vaccinations, rolling_count_people_vaccinated)
as
(
select deaths.continent, deaths.location, deaths.date, deaths.population, vaccinations.new_vaccinations,
sum(convert(int, vaccinations.new_vaccinations)) OVER (Partition by deaths.location Order By deaths.location,
deaths.Date) as rolling_count_people_vaccinated
from PortfolioProject..covid_data_deaths deaths
join PortfolioProject..covid_data_vaccinations vaccinations
	on deaths.location = vaccinations.location
	and deaths.date = vaccinations.date
where deaths.continent is not null
)

select *, round((rolling_count_people_vaccinated/population)*100, 3)
from people_vs_vacciantions


-- PERCENTAGE OF PEOPLE VACCINATED:
	--This might be a little inacurate because I didn't look into first and second doses, the point is to get the idea.
-- TEMPLATE TABLE

DROP TABLE if exists percentage_population_vaccinated
CREATE TABLE percentage_population_vaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
Rolling_People_Vaccinated numeric
)

SET ANSI_WARNINGS OFF
GO

Insert into percentage_population_vaccinated

select deaths.continent, deaths.location, deaths.date, deaths.population, vaccinations.new_vaccinations,
sum(convert(bigint, vaccinations.new_vaccinations)) OVER (Partition by deaths.location Order By deaths.location,
deaths.Date) as Rolling_People_Vaccinated
from PortfolioProject..covid_data_deaths deaths
join PortfolioProject..covid_data_vaccinations vaccinations
	on deaths.location = vaccinations.location
	and deaths.date = vaccinations.date


select *, (Rolling_People_Vaccinated/population)*100 as Percentage_Vaccinated
from percentage_population_vaccinated


-- CREATING VIEWS TO STORE DATA FOR LATER VISUALIZATIONS:

--DROP VIEW if exists percentage_population_vaccinated_view

CREATE VIEW percentage_population_vaccinated_view 
AS
select deaths.continent, deaths.location, deaths.date, deaths.population, vaccinations.new_vaccinations,
sum(convert(bigint, vaccinations.new_vaccinations)) OVER (Partition by deaths.location Order By deaths.location,
deaths.Date) as Rolling_People_Vaccinated
from PortfolioProject..covid_data_deaths deaths
join PortfolioProject..covid_data_vaccinations vaccinations
	on deaths.location = vaccinations.location
	and deaths.date = vaccinations.date
where deaths.continent is not null

Select *
from percentage_population_vaccinated


CREATE VIEW GlobalDeathsPercentage
AS
select sum(new_cases) as total_new_cases, sum(cast(new_deaths as int)) as total_new_deaths, round(sum(cast
(new_deaths as int))/sum(new_cases)*100, 3) as death_percentage
from PortfolioProject..covid_data_deaths
where continent is not null
--order by 1, 2
