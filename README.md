## Reservation Data Analysis Project Answers

### Project Data Points
**PROJECT REQUIREMENTS:**
- Completed project within 5 hours
- MySQL queries are added to project solutions folder
- Completed questionnaire in the root of the project directory under "README.md"
- Project pushed to GitHub

**TECHNOLOGY USED:**
- MySQL
- MySQL Workbench

**ASSUMPTIONS:**

- Members can book through either `mindbody_reservations` or `clubready_reservations`, therefore there could be duplicate information between reservation partners.
- Duplicate information can also exist within each reservation partner.
- The field **member_id** and **studio_key** is unique and can be used through different reservation partners.
- `NULL` in the **canceled_at** field meant that the individual **did not** cancel their reservation.
- `NULL` in the **checked_in_at** or **signed_in_at** field meant that the individual **did not** show up to their reservation.
- The following are fields that are equivalent across reservation partners:
    - **member_id** = **member_id**
    - **studio_key** = **studio_key**
    - **class_tag** = **class_tag**
    - **class_time_at** = **reserved_for**
    - **checked_in_at** = **signed_in_at**

<br/><br/>
**1.** Across all reservation partners for January & February, how many completed reservations occurred?
<br/><br/>

**ANSWER:**
``` mysql
SELECT
	COUNT(checked_in_at) as completed_reservations
FROM
	((SELECT
		member_id, studio_key, class_tag, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct
WHERE
	class_time_at BETWEEN '2018-01-01' AND '2018-03-01';
```
<br/><br/>
**2.** Which studio has the highest rate of reservation abandonment (did not cancel but did not check-in)?
<br/><br/>
**ANSWER:**
``` mysql
SELECT 
	studio_key, 1 - COUNT(checked_in_at) / COUNT(class_time_at) AS abandonment_rate
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, LTRIM(studio_key), class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct
WHERE
	canceled = 'f' OR (canceled = 't' AND checked_in_at IS NOT NULL)
GROUP BY
	studio_key
ORDER BY
	abandonment_rate DESC
LIMIT 1;
```
<br/><br/>
**3.** Which fitness area (i.e., tag) has the highest number of completed reservations for February?
<br/><br/>
**ANSWER:**
``` mysql
SELECT 
	class_tag, COUNT(checked_in_at) AS completed_reservation
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct
WHERE
	class_time_at BETWEEN '2018-02-01' AND '2018-03-01'
GROUP BY
	class_tag
ORDER BY
	completed_reservation DESC
LIMIT 1;
```
<br/><br/>
**4.** How many members completed at least 1 reservation and had no more than 1 cancelled reservation in January?
<br/><br/>
**ANSWER:**
``` mysql
SELECT
	COUNT(*) AS total_members
FROM
	(SELECT 
		member_id, SUM(CASE WHEN canceled = 't' AND checked_in_at IS NULL THEN 1 ELSE 0 END) AS canceled_count, COUNT(checked_in_at) AS completed_reservation_count
	FROM
		((SELECT
			member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
		FROM
			peerfit.mindbody_reservations AS mbr)
		UNION
		(SELECT
			member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
		FROM
			peerfit.clubready_reservations AS crr)) AS ct
	WHERE
		class_time_at BETWEEN '2018-01-01' AND '2018-02-01'
	GROUP BY
		member_id) AS final
WHERE
	canceled_count < 2 AND completed_reservation_count > 0;
```
<br/><br/>
**5.** At what time of day do most users book classes? Attend classes? (Morning = 7-11 AM, Afternoon = 12-4 PM, Evening = 5-10 PM)
<br/><br/>
**ANSWER (BOOKED):**
``` mysql
SELECT
	*
FROM	
((SELECT
	'7-11am' as time_range,
	SUM(CASE WHEN HOUR(class_time_at) BETWEEN 7 and 11 THEN 1 ELSE 0 END) AS number_of_people
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct)
UNION
(SELECT
	'12-4pm' as time_range,
    SUM(CASE WHEN HOUR(class_time_at) BETWEEN 12 and 16 THEN 1 ELSE 0 END) AS number_of_people
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct2)
UNION
(SELECT
	'5-10pm' as time_range,
    SUM(CASE WHEN HOUR(class_time_at) BETWEEN 17 and 22 THEN 1 ELSE 0 END) AS number_of_people
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct3)) AS final
ORDER BY
	number_of_people DESC 
LIMIT 1;
```
<br/><br/>
**ANSWER (ATTENDED):**
``` mysql
SELECT
	*
FROM	
((SELECT
	'7-11am' as time_range,
	SUM(CASE WHEN HOUR(checked_in_at) BETWEEN 7 and 11 THEN 1 ELSE 0 END) AS number_of_people
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct)
UNION
(SELECT
	'12-4pm' as time_range,
    SUM(CASE WHEN HOUR(checked_in_at) BETWEEN 12 and 16 THEN 1 ELSE 0 END) AS number_of_people
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct2)
UNION
(SELECT
	'5-10pm' as time_range,
    SUM(CASE WHEN HOUR(checked_in_at) BETWEEN 17 and 22 THEN 1 ELSE 0 END) AS number_of_people
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS ct3)) AS final
ORDER BY
	number_of_people DESC
LIMIT 1;
```
<br/><br/>
**ALTERNATE ANSWER (BOOKED):**
``` mysql
SELECT
	SUM(CASE WHEN HOUR(class_time_at) BETWEEN 7 and 11 THEN 1 ELSE 0 END) AS seven_eleven_am,
	SUM(CASE WHEN HOUR(class_time_at) BETWEEN 12 and 16 THEN 1 ELSE 0 END) AS twelve_four_pm,
	SUM(CASE WHEN HOUR(class_time_at) BETWEEN 17 and 22 THEN 1 ELSE 0 END) AS five_ten_pm
FROM
	 ((SELECT
		 member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	 FROM
		 peerfit.mindbody_reservations AS mbr)
	 UNION
	 (SELECT
		 member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	 FROM
		 peerfit.clubready_reservations AS crr)) AS ct;
```
<br/><br/>
**ALTERNATE ANSWER (ATTENDED):**
``` mysql
SELECT
	SUM(CASE WHEN HOUR(checked_in_at) BETWEEN 7 and 11 THEN 1 ELSE 0 END) AS seven_eleven_am,
	SUM(CASE WHEN HOUR(checked_in_at) BETWEEN 12 and 16 THEN 1 ELSE 0 END) AS twelve_four_pm,
	SUM(CASE WHEN HOUR(checked_in_at) BETWEEN 17 and 22 THEN 1 ELSE 0 END) AS five_ten_pm
FROM
	 ((SELECT
		 member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	 FROM
		 peerfit.mindbody_reservations AS mbr)
	 UNION
	 (SELECT
		 member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	 FROM
		 peerfit.clubready_reservations AS crr)) AS ct;
```
<br/><br/>
**6.** How many confirmed completed reservations did the member (ID) with the most reserved classes in February have?
<br/><br/>
**ANSWER:**
``` mysql
SELECT
	member_id, COUNT(class_time_at) AS reservation_count, COUNT(checked_in_at) AS completed_reservation_count
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) as ct
WHERE
	class_time_at BETWEEN '2018-02-01' AND '2018-03-01'
GROUP BY
	member_id
ORDER BY
	reservation_count DESC
LIMIT 1;
```
<br/><br/>
**7.** Write a query that unions the `mindbody_reservations` table and `clubready_reservations` table as cleanly as possible.
<br/><br/>
**ANSWER:**
``` mysql
SELECT
	t1.member_id, t1.studio_key, t1.class_tag, t1.canceled, t1.class_time_at, t1.checked_in_at, t4.viewed_at, t4.reserved_at,
    t2.studio_address_city, t2.studio_address_state, t2.studio_address_zip,
    t3.level
FROM
	((SELECT
		member_id, studio_key, class_tag, CASE WHEN canceled_at IS NULL THEN 'f' ELSE 't' END AS canceled, class_time_at, checked_in_at
	FROM
		peerfit.mindbody_reservations AS mbr)
	UNION
	(SELECT
		member_id, studio_key, class_tag, canceled, reserved_for AS class_time_at, signed_in_at AS checked_in_at
	FROM
		peerfit.clubready_reservations AS crr)) AS t1
LEFT JOIN
	(SELECT
		studio_key, class_tag, studio_address_street, studio_address_city, studio_address_state, studio_address_zip
	FROM
		peerfit.mindbody_reservations AS mbr
	GROUP BY
		studio_key, class_tag, studio_address_street, studio_address_city, studio_address_state, studio_address_zip) AS t2
ON (t1.studio_key = t2.studio_key OR (t1.studio_key IS NULL AND t2.studio_key IS NULL)) AND t1.class_tag = t2.class_tag
LEFT JOIN
	(SELECT
		member_id, studio_key, class_tag, instructor_full_name, level, reserved_for AS class_time_at
	FROM
		peerfit.clubready_reservations 
	GROUP BY
		member_id, studio_key, class_tag, instructor_full_name, level, reserved_for) as t3
ON t1.member_id = t3.member_id AND t1.studio_key = t3.studio_key AND t1.class_time_at = t3.class_time_at
LEFT JOIN
	(SELECT
		member_id, studio_key, viewed_at, reserved_at, class_time_at
	FROM
		peerfit.mindbody_reservations AS mbr) as t4
ON t1.member_id = t4.member_id AND t1.studio_key = t4.studio_key AND t1.class_time_at = t4.class_time_at;
```
<br/><br/>
### Project Discussion
**1.** What opportunities do you see to improve data storage and standardization for these datasets?
<br/><br/>
**ANSWER:**<br/>
There is an opportunity to separate the data into different tables. Separating the data into different tables **(see below)** will have the following benefits:

**GENERAL BENEFITS:**
- Builds consistency through standardization, therefore the data can be relied upon for different types of studies. (eg. CTR, A|B Testing for effectiveness of impressions, behavioral studies, etc...)
- Improve the overall data storage as information is not duplicated
- Increase efficiency for queries, only pulling necessary data from data warehouse

**SPECIFIC BENEFITS:**
- Having a **RESERVATION_PARTNER_ID** field with a `RESERVATION_PARTNER` table is beneficial when attempting to track which partner is the most effective, in addition to increased efficiency by not having duplicate information.
- Having a **CLASS_ID** field with a `CLASS` table is beneficial because we can now track popularity and increase efficiency by dropping partners that aren't producing results. In addition with this information, we can now build a recommendation engine (eg. collaborative filtering) to suggest which instructor, class or studio a user might like. Increasing activity of a user will be beneficial for employers (clients) and our company alike.
- Separating the tables `VIEW` and `RESERVE` allows us to do A|B testing on the poster that the user viewed to determine which poster is the most effective. This is to increase activity.

**TABLE:** `VIEW`
- VIEW_ID
- MEMBER_ID
- RESERVATION_PARTNER_ID
- CLASS_ID
- VIEWED_AT

**TABLE:** `RESERVE`
- RESERVE_ID
- VIEW_ID
- RESERVED_AT

**TABLE:** `CANCEL`
- CANCEL_ID
- RESERVE_ID
- CANCELLED_AT

**TABLE:** `CHECK_IN`
- CHECK_IN_ID
- RESERVE_ID
- CHECKED_IN_AT
    
**TABLE:** `MEMBER`
- MEMBER_ID
- MEMBER_NAME
- MEMBER_DEMOGRAPHICS_INFO (WHAT WE ARE ALLOWED TO COLLECT)
- EMPLOYER_ID

**TABLE:** `INSTRUCTOR`
- INSTRUCTOR_ID
- INSTRUCTOR_NAME
- INSTRUCTOR_DEMOGRAPHICS_INFO (WHAT WE ARE ALLOWED TO COLLECT)

**TABLE:** `RESERVATION_PARTNER`
- RESERVATION_PARTNER_ID
- RESERVATION_PARTNER_NAME
- RESERVATION_DEMOGRAPHICS_INFO (WHAT WE ARE ALLOWED TO COLLECT)

**TABLE:** `STUDIO`
- STUDIO_KEY
- STUDIO_NAME
- STUDIO_DEMOGRAPHICS_INFO (WHAT WE ARE ALLOWED TO COLLECT)

**TABLE:** `CLASS`
- CLASS_ID
- CLASS_TAG
- LEVEL
- CLASS_TIME_AT
- STUDIO_KEY
- INSTRUCTOR_ID

<br/><br/>
**2.** What forecasting opportunities do you see with a dataset like this and why?
<br/><br/>
**ANSWER:**<br/>
There are many forecasting opportunities, we can predict:
- Total bookings (Daily, Monthly, Quarterly, Semi-Annually, Annually)
- The number of people that will cancel their reservations
- Which reservation will be cancelled
- The number of people that will not attend their reservations
- Who will not show up to their reservation
- A classes that a users will like

These types of analysis and forecasting opportunities are very important to decision makers as it provides:
- Indicators for growth, whether a right decision was made
- A spotlight on areas of opportunity and ways to prioritize these concerns
- A tool to increase efficiency for budgeting and planning
- A way to increase/encourage activity for users

<br/><br/>
**3.** What other data would you propose we gather to make reporting/forecasting more robust and why?
<br/><br/>
**ANSWER:**<br/>
In addition to the way the tables are set up above I would want to collect the following:

**TABLE:** `STUDIO_BILL`
- STUDIO_BILL_ID
- STUDIO_KEY
- STUDIO_BILL_AMT
- STUDIO_BILL_DATE
    
**TABLE:** `STUDIO_PAID`
- STUDIO_PAID_ID
- STUDIO_BILL_ID
- STUDIO_PAID_AMT
- STUDIO_PAID_DATE

**TABLE:** `EMPLOYER`
- EMPLOYER_ID
- EMPLOYER_NAME
- EMPLOYER_DEMOGRAPHICS_INFO (WHAT WE ARE ALLOWED TO COLLECT)
    
**TABLE:** `EMPLOYER_BILL`
- EMPLOYER_BILL_ID
- EMPLOYER_ID
- EMPLOYER_BILL_AMT
- EMPLOYER_BILL_DATE
    
**TABLE:** `EMPLOYER_PAID`
- EMPLOYER_PAID_ID
- EMPLOYER_BILL_ID
- EMPLOYER_PAID_AMT
- EMPLOYER_PAID_DATE

The additional information will provide:
- An opportunity to look into "Customer Lifetime Value" **(CLTV)**. This is very important as it will indicate where the company should spend their time and efforts.
- Places a spotlight on areas of opportunity such as billing and collections. To increase the efficiency in collecting payments, which will effect the bottom-line.