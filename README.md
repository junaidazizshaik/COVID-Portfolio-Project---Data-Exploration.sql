SELECT *
FROM PortfolioProjects..CovidDeaths
where continent is not null
order by 3,4


--SELECT *
--FROM PortfoiloProjects..CovidDeaths
--order by 3,4

Select Location, date, total_cases, new_cases, total_deaths, population 
FROM PortfolioProjects..CovidDeaths
where continent is not null
order by 1,2 


--Looking at Total cases vs Total Deaths
--shows liklihood of dying if you contract in your country


SELECT Location, date, total_cases, total_deaths, 
    CASE 
        WHEN total_cases = 0 THEN 0
        ELSE (total_deaths / total_cases) * 100 
    END AS DeathPersentage
FROM 
   PortfolioProjects..CovidDeaths
where location like '%egypt%'
ORDER BY 
    1, 2


--Looking at Total cases vs 
--Shows What Percetage of Population Got Covid

SELECT Location, date, total_cases, population, 
    CASE 
        WHEN total_cases = 0 THEN 0
        ELSE (population / total_cases) * 100 
    END AS PersentagePopulationInfected
FROM 
   PortfolioProjects..CovidDeaths
--where location like '%egypt%'
ORDER BY 
    1, 2



--Looking at countries with the Highest Infection Rate Compared to the Population

Select Location, population,  MAX(total_cases) as HighestInfectionCount, MAX(total_cases/population)*100 as
    PercentagePopulationInfected
FROM PortfolioProjects..CovidDeaths
--where location like '%egypt%'
Group by Location, Population
order by PercentagePopulationInfected  desc


--Showing Countries with the Highest Death Count per Population

SELECT Location, CAST(MAX(Total_Deaths) AS INT) AS TotalDeathCount
FROM 
    PortfolioProjects..CovidDeaths
--WHERE Location LIKE '%egypt%'
where continent is not null
GROUP BY Location
ORDER BY TotalDeathCount DESC;



--Lets's Break Things Down by Continent 

--Showing Continents with the Highest Death COunt per Population

SELECT Continent, CAST(MAX(Total_Deaths) AS INT) AS TotalDeathCount
FROM 
    PortfolioProjects..CovidDeaths
--WHERE Location LIKE '%egypt%'
where continent is not null
GROUP BY Continent
ORDER BY TotalDeathCount DESC;


--Globla Number


SELECT 
    SUM(new_cases) AS total_cases,  
    SUM(CAST(new_deaths AS INT)) AS total_deaths,
    CASE 
        WHEN SUM(new_cases) = 0 THEN 0
        ELSE SUM(CAST(new_deaths AS INT)) / SUM(new_cases) * 100 
    END AS DeathPersentage
FROM 
    PortfolioProjects..CovidDeaths
WHERE 
    continent IS NOT NULL
GROUP BY  
    date
ORDER BY 
    1, 2;



--Looking at Total population vs Vaccinations

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
  ,SUM(CONVERT(int,vac.new_vaccinations)) OVER  ( Partition by dea.location order by dea .location, dea.date)
  as RollingPeopleVaccinated
  ,(RollingPeopleVaccinated/population)*100
FROM 
    PortfolioProjects..CovidDeaths dea
Join
   PortfolioProjects..CovidVaccinations vac
   On dea.location = vac.location
   and dea.date = vac.date
WHERE dea.continent is not null 
order by 2, 3


--USE CTE 


WITH Popvsvac (Continent, Location, Date, Population, New_Vaccination, RollingPeopleVaccinated)
AS
(
    SELECT 
        dea.continent, 
        dea.location, 
        dea.date, 
        dea.population, 
        vac.new_vaccinations,
        SUM(CONVERT(bigint, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated
    FROM 
        PortfolioProjects..CovidDeaths dea
    JOIN
        PortfolioProjects..CovidVaccinations vac
        ON dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL 
)
SELECT 
    *,
    (RollingPeopleVaccinated/CONVERT(float, Population))*100 AS PercentageVaccinated
FROM 
    Popvsvac;




--TEMP TABLE 
DROP table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
RollingPeopleVaccinated numeric
)

    Insert into #PercentPopulationVaccinated
    SELECT 
        dea.continent, 
        dea.location, 
        dea.date, 
        dea.population, 
        vac.new_vaccinations,
        SUM(CONVERT(bigint, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated
    FROM 
        PortfolioProjects..CovidDeaths dea
    JOIN
        PortfolioProjects..CovidVaccinations vac
        ON dea.location = vac.location
        AND dea.date = vac.date
    --WHERE dea.continent IS NOT NULL 

SELECT 
    *,
    (RollingPeopleVaccinated/CONVERT(float, Population))*100 AS PercentageVaccinated
FROM 
    #PercentPopulationVaccinated



--Creating View to store data for later viualizations

Create View PercentPopulationVaccinated as 
   SELECT 
        dea.continent, 
        dea.location, 
        dea.date, 
        dea.population, 
        vac.new_vaccinations,
        SUM(CONVERT(bigint, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated
    FROM 
        PortfolioProjects..CovidDeaths dea
    JOIN
        PortfolioProjects..CovidVaccinations vac
        ON dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL 
	--order by 2,3


Select *
From PercentPopulationVaccinated
