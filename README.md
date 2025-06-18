/* Beginner

-- Q1.who is the senior most employee based on job title
-- select *from employee
-- order by title desc
-- limit 1;

-- Q2.which countries have the most invoices?
select *from invoice
-- select count(*)as c, billing_country
-- from invoice 
-- group by billing_country
-- order by c desc;

-- Q3. what are the top 3 values pf total values?

-- select total from invoice
-- order by total desc 
-- limit 3;

/* Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. 
Write a query that returns one city that has the highest sum of invoice totals. 
Return both the city name & sum of all invoice totals */
select *from invoice
select sum (total) as invoice_total,billing_city from invoice
group by billing_city
order by invoice_total desc

/* Q5: Who is the best customer? The customer who has spent the most money will be declared the best customer. 
Write a query that returns the person who has spent the most money.*/
select *from customer

-- select customer.customer_id,customer.first_name,customer,last_name,sum(total) as total
-- from customer
-- join invoice on customer.customer_id=invoice.customer_id
-- group by customer.customer_id
-- order by total desc
-- limit 1;

/* Question Set 2 - Moderate */

/* Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
Return your list ordered alphabetically by email starting with A. */

/*Method 1 */
-- select *from invoice_line
-- select *from genre

-- select email,first_name,last_name
-- from customer
-- join invoice on customer.customer_id=invoice.customer_id
-- join invoice_line on invoice.invoice_id=invoice_line.invoice_id
-- where track_id in(
-- select track_id from track
-- JOIN genre ON track.genre_id = genre.genre_id
-- 	WHERE genre.name LIKE 'Rock'
-- )
-- ORDER BY email;
-- )
/* Method 2 */
-- select *from track
-- SELECT DISTINCT email AS Email,first_name AS FirstName, last_name AS LastName, genre.name AS Name
-- FROM customer
-- JOIN invoice ON invoice.customer_id = customer.customer_id
-- JOIN invoice_line ON invoice_line.invoice_id = invoice.invoice_id
-- JOIN track ON track.track_id = invoice_line.track_id
-- JOIN genre ON genre.genre_id = track.genre_id
-- WHERE genre.name = 'Rock'
-- ORDER BY email;

/* Q2: Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

-- SELECT artist.artist_id, artist.name,COUNT(artist.artist_id) AS number_of_songs
-- FROM track
-- JOIN album ON album.album_id = track.album_id
-- JOIN artist ON artist.artist_id = album.artist_id
-- JOIN genre ON genre.genre_id = track.genre_id
-- WHERE genre.name LIKE 'Rock'
-- GROUP BY artist.artist_id
-- ORDER BY number_of_songs DESC
-- LIMIT 10;

select *from track
select * from invoice_line
/* Q3: Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. */
-- select name,milliseconds 
-- from track 
-- where milliseconds>(
-- select avg(milliseconds)from track as avg_track_length
-- )

-- order by milliseconds desc


/* Question Set 3 - Advance */

/* Q1: Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent */

/* Steps to Solve: First, find which artist has earned the most according to the InvoiceLines. Now use this artist to find 
which customer spent the most on this artist. For this query, you will need to use the Invoice, InvoiceLine, Track, Customer, 
Album, and Artist tables. Note, this one is tricky because the Total spent in the Invoice table might not be on a single product, 
so you need to use the InvoiceLine table to find out how many of each product was purchased, and then multiply this by the price
for each artist. */
with best_selling as(
select artist.artist_id as artist_id, artist.name as artist_name,
sum(invoice_line.unit_price*invoice_line.quantity)as total_sales
from invoice_line
join track on track.track_id=invoice_line.track_id
join album on album.album_id=track.album_id
join artist on artist.artist_id=album.artist_id
group by 1
order by 1
)

select c.customer_id,c.first_name,c.last_name,bsa.artist_name,
sum(il.unit_price*il.quantity) as amount_spent
from invoice i
join customer  c on c.customer_id=i.customer_id
join invoice_line il on il.invoice_id=i.invoice_id
join track t on t.track_id=il.track_id
join album alb on alb.album_id = t.album_id
join best_selling bsa on bsa.artist_id = alb.artist_id
group by 1,2,3,4
order by 5 desc

/* Q2: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
the maximum number of purchases is shared return all Genres. */

/* Steps to Solve:  There are two parts in question- first most popular music genre and second need data at country level. */
with popular_genre as(
select count(invoice_line.quantity) as purchases, customer.country, genre.name,genre.genre_id,
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
from invoice_line
join invoice on invoice.invoice_id=invoice_line.invoice_id
join customer on customer.customer_id=invoice.customer_id
join track on track.track_id=invoice_line.track_id
join genre on  genre.genre_id=track.genre_id
group by 2,3,4
order by 2 asc, 1 desc
)
select *from popular_genre where RowNo<=1

/* Q3: Write a query that determines the customer that has spent the most on music for each country. 
Write a query that returns the country along with the top customer and how much they spent. 
For countries where the top amount spent is shared, provide all customers who spent this amount. */

/* Steps to Solve:  Similar to the above question. There are two parts in question- 
first find the most spent on music for each country and second filter the data for respective customers. */
/* Method 1: using CTE */

WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1


/* Method 2: Using Recursive */

WITH RECURSIVE 
	customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 2,3 DESC),

	country_max_spending AS(
		SELECT billing_country,MAX(total_spending) AS max_spending
		FROM customter_with_country
		GROUP BY billing_country)

SELECT cc.billing_country, cc.total_spending, cc.first_name, cc.last_name, cc.customer_id
FROM customter_with_country cc
JOIN country_max_spending ms
ON cc.billing_country = ms.billing_country
WHERE cc.total_spending = ms.max_spending
ORDER BY 1;


![image](https://github.com/user-attachments/assets/0e048bce-1766-410e-af80-d5c197ca999c)
