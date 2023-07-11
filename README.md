# prescriber_project

    --1. a. Which prescriber had the highest total number of claims (totaled over all drugs)? Report the npi and the total number of claims.

SELECT npi, SUM(total_claim_count) AS total_claims

FROM prescription

GROUP BY npi


ORDER BY total_claims DESC

LIMIT 1;

    --99707

    --b. Repeat the above, but this time report the nppes_provider_first_name, nppes_provider_last_org_name,  specialty_description, and the total number of claims.

SELECT SUM(p2.total_claim_count) AS claim_count, p2.npi, p1.nppes_provider_last_org_name AS last_name, p1.nppes_provider_first_name AS first_name, p1.specialty_description 

FROM prescriber AS p1
   
INNER JOIN prescription AS p2
    
ON p2.npi = p1.npi
	
 GROUP BY p1.nppes_provider_last_org_name, p1.nppes_provider_first_name, p2.npi, p1.specialty_description
	
 ORDER BY claim_count DESC;

    --Bruce Pendley

    --2. a. Which specialty had the most total number of claims (totaled over all drugs)?

SELECT SUM(p.total_claim_count) AS total, prescriber.specialty_description

FROM prescription AS p

INNER JOIN prescriber

ON p.npi = prescriber.npi

GROUP BY prescriber.specialty_description

ORDER BY total DESC;

    --Family Practice with 9752347

    --b. Which specialty had the most total number of claims for opioids?

SELECT SUM(p.total_claim_count) AS total, prescriber.specialty_description, drug.opioid_drug_flag

FROM prescription AS p

INNER JOIN prescriber

ON p.npi = prescriber.npi

INNER JOIN drug

ON p.drug_name = drug.drug_name

WHERE drug.opioid_drug_flag = 'Y'

GROUP BY prescriber.specialty_description, drug.opioid_drug_flag

ORDER BY total DESC;

    --Nurse Practitioner with 900845

    --3. a. Which drug (generic_name) had the highest total drug cost?

SELECT drug.generic_name AS drug, SUM(p.total_drug_cost) AS total_cost

FROM drug

INNER JOIN prescription AS p

ON drug.drug_name = p.drug_name

GROUP BY drug

ORDER BY total_cost DESC;

    --"INSULIN GLARGINE,HUM.REC.ANLOG"	104264066.35

    --b. Which drug (generic_name) has the hightest total cost per day? **Bonus: Round your cost per day column to 2 decimal places. Google ROUND to see how this works.**

SELECT generic_name, ROUND(SUM(total_drug_cost) / SUM(total_day_supply), 2)::MONEY AS cost_per_day

FROM drug

JOIN prescription ON drug.drug_name = prescription.drug_name

GROUP BY generic_name

ORDER BY cost_per_day DESC

LIMIT 10;

    --"C1 ESTERASE INHIBITOR"	"$3,495.22"

    --4. a. For each drug in the drug table, return the drug name and then a column named 'drug_type' which says 'opioid' for drugs which have opioid_drug_flag = 'Y', says 'antibiotic' for those drugs which have antibiotic_drug_flag = 'Y', and says 'neither' for all other drugs.

SELECT DISTINCT drug_name,

CASE 
	
 WHEN opioid_drug_flag = 'Y' THEN 'Opioid'
	
 WHEN antibiotic_drug_flag = 'Y' THEN 'Antibiotic'
	
 ELSE 'Neither' 
	
 END AS drug_type

FROM drug;

    --b. Building off of the query you wrote for part a, determine whether more was spent (total_drug_cost) on opioids or on antibiotics. Hint: Format the total costs as MONEY for easier comparision.

SELECT

CASE WHEN opioid_drug_flag = 'Y' THEN 'Opioid'
	
 WHEN antibiotic_drug_flag = 'Y' THEN 'Antibiotic'
	
 ELSE 'Neither' 
	
 END AS drug_type,
	
 SUM(total_drug_cost)::MONEY AS total_cost

FROM drug AS d

INNER JOIN prescription AS p USING(drug_name)

GROUP BY drug_type

ORDER BY total_cost DESC;

    --"Neither"	"$2,972,698,710.23"
    --"Opioid"	"$105,080,626.37"
    --"Antibiotic"	"$38,435,121.26"

    --5. a. How many CBSAs are in Tennessee? **Warning:** The cbsa table contains information for all states, not just Tennessee.

SELECT COUNT(c.cbsa),f.state

FROM cbsa AS c

INNER JOIN fips_county AS f

ON c.fipscounty = f.fipscounty

WHERE f.state = 'TN'

GROUP BY f.state
    --42

    --b. Which cbsa has the largest combined population? Which has the smallest? Report the CBSA name and total population.

SELECT c.cbsaname AS county, SUM(p.population) AS population

FROM cbsa AS c

INNER JOIN population AS p

ON c.fipscounty = p.fipscounty

GROUP BY c.cbsaname

ORDER BY SUM(p.population) DESC;

    --"Nashville-Davidson--Murfreesboro--Franklin, TN" - most
    --"Morristown, TN" - least

    --c. What is the largest (in terms of population) county which is not included in a CBSA? Report the county name and population

SELECT f.county, p.population, f.fipscounty

FROM fips_county AS f

INNER JOIN population AS p

ON f.fipscounty = p.fipscounty

WHERE f.fipscounty NOT IN 

(SELECT fipscounty

FROM cbsa

ORDER BY cbsa)

ORDER BY p.population DESC

    --Sevier

    --6. a. Find all rows in the prescription table where total_claims is at least 3000. Report the drug_name and the total_claim_count.

SELECT p.total_claim_count, d.drug_name

FROM prescription AS p

INNER JOIN drug AS d

ON p.drug_name = d.drug_name

WHERE p.total_claim_count >= 3000

ORDER BY p.total_claim_count DESC;

    --b. For each instance that you found in part a, add a column that indicates whether the drug is an opioid.

SELECT p.total_claim_count, d.drug_name, d.opioid_drug_flag

FROM prescription AS p

INNER JOIN drug AS d

ON p.drug_name = d.drug_name

WHERE p.total_claim_count >= 3000

ORDER BY p.total_claim_count DESC;

    --c. Add another column to you answer from the previous part which gives the prescriber first and last name associated with each row.

SELECT p.total_claim_count, d.drug_name, d.opioid_drug_flag, prescriber.nppes_provider_first_name AS first_name, prescriber.nppes_provider_last_org_name AS last_name

FROM prescription AS p

INNER JOIN drug AS d

ON p.drug_name = d.drug_name

INNER JOIN prescriber 

ON prescriber.npi = p.npi

WHERE p.total_claim_count >= 3000

ORDER BY p.total_claim_count DESC;
