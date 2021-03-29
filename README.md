# sol

alter proc lbrms.usp_PlaceOrder
(
	@BookID INT,
	@StudentID INT
)
AS
BEGIN
	IF(@BookID IS NOT NULL AND @StudentID IS NOT NULL) AND ((LEN(@BookID)!=0) AND (LEN(@StudentID)!=0))
			
			BEGIN
				BEGIN TRY
				--cHECKING IF HAD REQUESTED IT MORE THAN ONCE IN ONE CATEGORY
									-- requesting category
					-- already requested or issued categories

					IF EXISTS (select BookID from lbrms.Tr_BookTranscationsHeader  				 
					where StudentID = @StudentID and BookID = @BookID and (Status = 'Requested'  or Status = 'Issued'))
					BEGIN
						SELECT 'You cannot request for the same book twice'
					END

					ELSE IF EXISTS (SELECT CategoryID FROM lbrms.Ma_Book WHERE BookID = @BookID AND CategoryID IN (
					select b.CategoryID from lbrms.Tr_BookTranscationsHeader tbh
					join lbrms.Ma_Book b
					on tbh.BookID = b.BookID
					where StudentID=1 and (tbh.Status = 'Requested' or tbh.Status = 'Issued') ))
					BEGIN
						SELECT 'You have already booked from this category, Request Denied'
					END
					ELSE
					BEGIN
					if exists (
						select  Quantity from lbrms.Ma_Book
						where BookID = @BookID and Quantity > 0)
						begin
						insert into lbrms.Tr_BookTranscationsHeader(BookID,StudentID,RequestedDate,Status) values (@BookID,@StudentID,GETDATE(),'Requested')
						SELECT 'Request Success'
						end
						else
						begin
								SELECT 'Quantity out of Stock, Request Denied'

						end
					END




				END TRY

				BEGIN CATCH
				END CATCH
			END
END
exec lbrms.usp_PlaceOrder 6,1

**********************************************
--****************search , reuest and place order*****************************************************

alter proc lbrms.usp_PlaceOrder
(
	@BookID INT,
	@StudentID INT
)
AS
BEGIN
	IF(@BookID IS NOT NULL AND @StudentID IS NOT NULL) AND ((LEN(@BookID)!=0) AND (LEN(@StudentID)!=0))
			
			BEGIN
				BEGIN TRY
				--cHECKING IF HAD REQUESTED IT MORE THAN ONCE IN ONE CATEGORY
									-- requesting category
					-- already requested or issued categories

					IF EXISTS (select BookID from lbrms.Tr_BookTranscationsHeader  				 
					where StudentID = @StudentID and BookID = @BookID and (Status = 'Requested'  or Status = 'Issued'))
					BEGIN
						SELECT 'You cannot request for the same book twice'
					END

					ELSE IF EXISTS (SELECT CategoryID FROM lbrms.Ma_Book WHERE BookID = @BookID AND CategoryID IN (
					select b.CategoryID from lbrms.Tr_BookTranscationsHeader tbh
					join lbrms.Ma_Book b
					on tbh.BookID = b.BookID
					where StudentID=1 and (tbh.Status = 'Requested' or tbh.Status = 'Issued') ))
					BEGIN
						SELECT 'You have already booked from this category, Request Denied'
					END
					ELSE
					BEGIN
					if exists (
						select  Quantity from lbrms.Ma_Book
						where BookID = @BookID and Quantity > 0)
						begin
						insert into lbrms.Tr_BookTranscationsHeader(BookID,StudentID,RequestedDate,Status) values (@BookID,@StudentID,GETDATE(),'Requested')
						SELECT 'Request Success'
						end
						else
						begin
								SELECT 'Quantity out of Stock, Request Denied'

						end
					END




				END TRY

				BEGIN CATCH
				END CATCH
			END
END
exec lbrms.usp_PlaceOrder 6,1
--*************************************************************************************


select * from lbrms.Ma_Book
select * from lbrms.Ma_Student
select * from lbrms.Tr_BookTranscationsHeader
insert into lbrms.Tr_BookTranscationsHeader(BookID,StudentID,RequestedDate,Status) values (2,2,GETDATE(),'Requested'),(3,2,GETDATE(),'Requested')

delete from lbrms.Tr_BookTranscationsHeader
where IssueID = 5

select * from lbrms.Ma_Student
insert into lbrms.Ma_Student(Name,Gender,EmailID,Password) values ('a','F','a@gmail.com','1')

update lbrms.Ma_Book 
set Quantity = 1
where BookID = 3



select * from lbrms.Ma_Book
INSERT INTO lbrms.Ma_Book(ISBN,Title,Author,CategoryID,Quantity,DESCRIPTION) VALUES
(9875632587416,'Harry Potter','JK Rowling',3,5,'Animation')





--****************generate Admin overdue report********************************
ALTER PROC lbrms.usp_GenerateOverDue
AS
BEGIN


WITH cte AS(
	 select StudentID,BookID, (DATEDIFF( DAY, DueDate,ReturnDate))*5 AS 'Penalty' from lbrms.Tr_BookTranscationsHeader 
	 WHERE DATEDIFF( DAY, DueDate,ReturnDate)> 0
	 	 UNION
	 SELECT StudentID,BookID,(DATEDIFF( DAY, DueDate,GETDATE()))*5 AS 'Penalty'  FROM lbrms.Tr_BookTranscationsHeader 
	 WHERE ReturnDate IS NULL AND DATEDIFF( DAY, DueDate,GETDATE())> 0 
	 )
	 SELECT StudentID, SuM(Penalty) AS 'TotalPenalty' FROM cte
	 Group by StudentID

END
GO

exec lbrms.usp_GenerateOverDue

--**************************************************************************************8


SELECT * FROM lbrms.Tr_BookTranscationsHeader
UPDATE lbrms.Tr_BookTranscationsHeader
SET ReturnDate = '2021-04-06 01:39:00'
WHERE IssueID= 3







--***************Student Over Due AReport*****************************************************

ALTER PROC lbrms.usp_StudentOverDue
(
	@StdID INT
)
AS
BEGIN
	
	--BOOKID DUEDATE RETURNEDDATE   PENATLY
	
	--SELECT BookID,DueDate,ReturnDate, CASE WHEN a.ReturnDate IS NOT NULL THEN (DATEDIFF( DAY, DueDate,ReturnDate))*5 ELSE (DATEDIFF( DAY, DueDate,GETDATE()))*5 END  AS 'Penalty'  FROM lbrms.Tr_BookTranscationsHeader a
	 --WHERE DATEDIFF( DAY, DueDate,ReturnDate)> 0 AND StudentID = 2 OR (ReturnDate IS NULL AND DATEDIFF( DAY, DueDate,GETDATE())> 0)
	 
	 SELECT BookID,DueDate,ReturnDate, (DATEDIFF( DAY, DueDate,ReturnDate))*5  AS 'Penalty'  FROM lbrms.Tr_BookTranscationsHeader a
	 WHERE DATEDIFF( DAY, DueDate,ReturnDate)> 0 AND StudentID = @StdID
	 UNION
	 SELECT BookID,DueDate,ReturnDate,(DATEDIFF( DAY, DueDate,GETDATE()))*5 AS 'Penalty'  FROM lbrms.Tr_BookTranscationsHeader 
	 WHERE ReturnDate IS NULL AND DATEDIFF( DAY, DueDate,GETDATE())> 0 AND StudentID = @StdID

END
GO



EXEC lbrms.usp_StudentOverDue 2

--**************************************************************************************8


--SELECT * FROM lbrms.Tr_BookTranscationsHeader 
--where ReturnDate is null


--update lbrms.Tr_BookTranscationsHeader 
--set DueDate = '2021-03-27 11:02:56', ReturnDate = '2021-03-28 11:25:23'
--where IssueID = 4



--SELECT * FROM lbrms.Tr_BookTranscationsHeader
--select RegisteredDate from lbrms.Ma_Student
--UPDATE lbrms.Tr_BookTranscationsHeader
--SET DueDate = '2021-03-25 12:32:00', ReturnDate = '2021-03-25 12:33:00'
--where StudentID = 1



--int sum = 0;
----summing all overdues
--while()
--{
-- int penalty =	reder['Penalty'];
--    sum += prenalty

--}

-- SELECT DateDiff(day, GetDate(), CASE WHEN RegisteredDate > (SELECT CASE WHEN MAX(RequestedDate)>MAX(ReturnDate) THEN  MAX(RequestedDate) ELSE MAX(ReturnDate) END AS 'MaxVal' FROM lbrms.Tr_BookTranscationsHeader WHERE StudentID = a.StudentID) THEN RegisteredDate ELSE (SELECT CASE WHEN MAX(RequestedDate)>MAX(ReturnDate) THEN  MAX(RequestedDate) ELSE MAX(ReturnDate) END AS 'MaxVal' FROM lbrms.Tr_BookTranscationsHeader WHERE StudentID = a.StudentID) END) FROM lbrms.Ma_Student a

--select * from lbrms.Tr_BookTranscationsHeader 

--insert into lbrms.Ma_Student(Name,Gender,EmailID,Password) values ('b','F','b@gmail.com','2')





--select * from lbrms.Ma_Student b
--LEFT OUTER JOIN lbrms.Tr_BookTranscationsHeader a
--ON b.StudentID = a.StudentID
--WHERE RequestedDate IS NULL AND DATEDIFF(Day,b.RegisteredDate,GETDATE()) > 15

--SELECT * FROM lbrms.Ma_Student b
--WHERE Re


--Update lbrms.Ma_Student
--Set RegisteredDate = '2021-03-10'
--Where StudentID =4

--SELECT * FROM lbrms.Tr_BookTranscationsHeader WHERE Status = 'Requested'





--**********************8*****Procedure to view issued book fr student****************************
ALTER PROC lbrms.usp_StudentViewIssuedBooks
(
	@StudendID INT
)
AS
BEGIN
	IF ( @StudendID IS NOT NULL) AND (LEN(@StudendID)!=0)
		BEGIN
			BEGIN TRY
				SELECT StudentID,BookID, IssuedDate,Status, IssueID FROM lbrms.Tr_BookTranscationsHeader WHERE Status = 'Issued' AND StudentID = @StudendID
			END TRY

			BEGIN CATCH
					INSERT INTO[lbrms].[ErrorLog]
					([ErrorTime],[UserName],[ErrorNumber],[ErrorSeverity],[ErrorState],[ErrorProcedure],[ErrorLine],[ErrorMessage])
                    VALUES(GETDATE(), SUSER_SNAME(), ERROR_NUMBER(), ERROR_SEVERITY(), ERROR_STATE(), ERROR_PROCEDURE(), ERROR_LINE(), ERROR_MESSAGE())
                    SELECT ERROR_MESSAGE()
			END CATCH
		END
END

EXEC lbrms.usp_StudentViewIssuedBooks 1
--**************************************************************************************8



--********************************Return BOkk ADmin*****************************
--CREATE PROC lbrms.usp_AdminAprroveReturn
--(
--	@IssueID INT
--	--@StudentID INT

--)
--AS
--BEGIN
--	IF (@IssueID IS NOT NULL AND @StudentID IS NOT NULL) AND ((LEN(@StudentID)!=0) AND (LEN(@IssueID)!=0))
--		BEGIN
--			IF EXISTS (SELECT * FROM lbrms.Tr_BookTranscationsHeader WHERE IssueID = @IssueID)
--			BEGIN TRY
--				IF EXISTS 
--				(
--					SELECT BookID, StudentID,Status FROM lbrms.Tr_BookTranscationsHeader 
--					WHERE IssueID = @IssueID AND StudentID = @StudentID AND Status = 'Requested')
--					BEGIN
--						UPDATE lbrms.Tr_BookTranscationsHeader
--						SET ReturnDate = GETDATE()

--						INSERT INTO lbrms.Tr_BookTranscationsHeader(ReturnDate,Status) VALUES (GETDATE(),'Returned')
--					END

				
--			END TRY

--			BEGIN CATCH
			
--			END CATCH

--		END
--END

select * from lbrms.Tr_BookTranscationsHeader
--**********************************************************************************************






--*************************Admin ISSING BOOK******************8***************
ALTER PROC lbrms.usp_AdminIssuingBook
(
	@IssID INT  
)
AS
    BEGIN
        IF EXISTS (SELECT * FROM lbrms.Tr_BookTranscationsHeader
            where IssueID = @IssID )
        BEGIN

			IF EXISTS (SELECT Quantity FROM lbrms.Ma_Book 
				WHERE (Quantity > 0) AND BookID = (SELECT  BookID FROM lbrms.Tr_BookTranscationsHeader WHERE IssueID = @IssID )
			)
			BEGIN

				UPDATE lbrms.Tr_BookTranscationsHeader
				SET IssuedDate = GetDate(), DueDate = (GetDate()+7), Status = 'Issued'
				WHERE IssueID = @IssID

				UPDATE lbrms.Ma_Book
				SET Quantity = Quantity -1
				WHERE  BookID = (SELECT BookID FROM lbrms.Tr_BookTranscationsHeader WHERE IssueID = @IssID )

			END


        END
    END

exec lbrms.usp_AdminIssuingBook 10

--********************************************************************************************8

--select* from lbrms.Tr_BookTranscationsHeader WHERE BookID = 3
--select * from lbrms.Ma_Book where BookID = 5

--select b.Title, bth.BookID,  bth.IssueID, b.Quantity from lbrms.Tr_BookTranscationsHeader bth
--join lbrms.Ma_Book B
--on B.BookID = bth.BookID
--WHERE bth.Status = 'Requested'


--*************************usp_ViewRequestedBookRecords************************
CREATE PROC lbrms.usp_ViewRequestedBookRecords
AS
BEGIN
	
	select b.Title, bth.BookID,  bth.IssueID, b.Quantity from lbrms.Tr_BookTranscationsHeader bth
join lbrms.Ma_Book B
on B.BookID = bth.BookID
WHERE bth.Status = 'Requested'

END

--**************************************************************************************




--*******************************sTUDENT RETURING BOOK**************************
ALTER PROC lbrms.usp_StudentBookReturn
(
	@IssueID INT

)
AS
BEGIN
	IF (@IssueID IS NOT NULL) AND ((LEN(@IssueID)!=0))
		BEGIN
			IF EXISTS (SELECT * FROM lbrms.Tr_BookTranscationsHeader WHERE IssueID = @IssueID)
			BEGIN TRY
				IF EXISTS 
				(
					SELECT * FROM lbrms.Tr_BookTranscationsHeader 
					WHERE IssueID = @IssueID AND Status = 'Issued')
					BEGIN

						UPDATE lbrms.Tr_BookTranscationsHeader
						SET ReturnDate = GETDATE(), Status = 'Returned'
						WHERE IssueID = 2

						UPDATE lbrms.Ma_Book
						SET Quantity = Quantity + 1
						WHERE BookID = (SELECT BookID FROM lbrms.Tr_BookTranscationsHeader WHERE IssueID = @IssueID )
					END

				
			END TRY

			BEGIN CATCH
			
			END CATCH

		END
END

exec lbrms.usp_StudentBookReturn 2


