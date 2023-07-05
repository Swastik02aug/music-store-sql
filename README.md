# music-store-sql

                                                                   /* Question Set 1 - Easy */

	   

/* Q1: Who is the senior most employee based on job title? */



select employee_id,first_name,last_name,title from employee
order by levels DESC
limit 1



/* Q2: Which countries have the most Invoices? */



select billing_country ,count(invoice_id)as no_of_invoice from invoice
group by billing_country
order by no_of_invoice DESC
limit 1




/* Q3: What are top 3 values of total invoice? */



select total from invoice
order by total DESC
limit 3



/* Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money.
Write a query that returns one city that has the highest sum of invoice totals.
Return both the city name & sum of all invoice totals */



select billing_city, sum(total) as invoice_total
from invoice 
group by 1
order by 2  desc
limit 1



/* Q5: Who is the best customer? The customer who has spent the most money will be declared the best customer.
Write a query that returns the person who has spent the most money.*/



select c.customer_id,c.first_name,c.last_name,sum(i.total) as total_spent from invoice i
join customer c on c.customer_id=i.customer_id
group by 1
order by 4 DESC
limit 1




                                            /* Question Set 2 - Moderate */


  

/* Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners.
Return your list ordered alphabetically by email starting with A. */



/*Method 1 */



select c.email,c.first_name,c.last_name from customer c
join invoice i on i.customer_id=c.customer_id
join invoice_line il on il.invoice_id=i.invoice_id
join track t on t.track_id=il.track_id
join genre g on g.genre_id=t.genre_id
where g.name  like 'Rock'
order by 1
limit 1




/* Q2: Let's invite the artists who have written the most rock music in our dataset.
Write a query that returns the Artist name and total track count of the top 10 rock bands. */




select a.artist_id,a.name,count(a.artist_id) as no_of_songs from track t
join album alb on alb.album_id=t.album_id
join artist a on a.artist_id=alb.artist_id
join genre g on g.genre_id=t.genre_id
where g.name like 'Rock'
group by 1
order by 3 desc
limit 10



/* Q3: Return all the track names that have a song length longer than the average song length.
Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. */



select name ,milliseconds from track
where milliseconds>(select avg(milliseconds) as average from track)
order by milliseconds desc




                                                  /* Question Set 3 - Advance */


 

/* Q1: Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent */




/* Steps to Solve: First, find which artist has earned the most according to the InvoiceLines. Now use this artist to find
which customer spent the most on this artist. For this query, you will need to use the Invoice, InvoiceLine, Track, Customer,
Album, and Artist tables. Note, this one is tricky because the Total spent in the Invoice table might not be on a single product,
so you need to use the InvoiceLine table to find out how many of each product was purchased, and then multiply this by the price
for each artist. */



WITH expensive_aritist as(
	select a.artist_id,a.name as artist_name ,sum(il.unit_price*il.quantity) as spent  from artist a
	join album alb on alb.artist_id=a.artist_id
	join track t on t.album_id =alb.album_id
	join invoice_line il on il.track_id=t.track_id
	group by 1
	order by 3 desc
	limit 1
)
select c.customer_id,c.first_name,c.last_name ,ea.artist_name, sum(il.unit_price*il.quantity)as customer_spent 
from customer c
join invoice i on i.customer_id=c.customer_id
join invoice_line il on il.invoice_id=i.invoice_id
join track t on t.track_id=il.track_id
join album alb on alb.album_id=t.album_id
join expensive_aritist ea on ea.artist_id=alb.artist_id
group by 1,2,3,4
order by customer_spent desc





/* Q2: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre
with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where
the maximum number of purchases is shared return all Genres. */





/* Steps to Solve: There are two parts in question- first most popular music genre and second need data at country level. */



/* Method 1: Using CTE */



WITH most_popular_genre as(
select customer.country,genre.name,genre.genre_id,count(invoice_line.quantity)as no_of_purchase ,
row_number() over(partition by customer.country order by count(invoice_line.quantity) desc)as rowno
from invoice_line
join invoice on invoice.invoice_id=invoice_line.invoice_id
join customer on customer.customer_id=invoice.customer_id
join track on track.track_id=invoice_line.track_id
join genre on genre.genre_id=track.genre_id
group by 1,2,3
order by 1 asc,4 desc
)
select *from most_popular_genre 
where rowno<=1;





/* Q3: Write a query that determines the customer that has spent the most on music for each country.
Write a query that returns the country along with the top customer and how much they spent.
For countries where the top amount spent is shared, provide all customers who spent this amount. */



/* Steps to Solve: Similar to the above question. There are two parts in question-
first find the most spent on music for each country and second filter the data for respective customers. */




/* Method 1: using CTE */




with customer_with_country as(
select customer.customer_id,first_name,last_name,billing_country,sum(total)as total_spend,
row_number() over(partition by billing_country order by sum(total)desc)as rowno
		  from invoice
		  join customer on customer.customer_id=invoice.customer_id
		  group by 1,2,3,4
		  order by 4 asc ,5 desc
		 )
	select* from customer_with_country where rowno<=1;


 

/* Method 2: Using Recursive */

WITH RECURSIVE customter_with_country AS ( 
	SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending 
	FROM invoice JOIN customer ON customer.customer_id = invoice.customer_id 
	GROUP BY 1,2,3,4 ORDER BY 2,3 DESC
),

country_max_spending AS( 
	SELECT billing_country,MAX(total_spending) AS max_spending 
	FROM customter_with_country 
	GROUP BY billing_country) 
	SELECT cc.billing_country, cc.total_spending, cc.first_name, cc.last_name, cc.customer_id 
	FROM customter_with_country cc 
	JOIN country_max_spending ms ON cc.billing_country = ms.billing_country 
	WHERE cc.total_spending = ms.max_spending 
	ORDER BY 1;
