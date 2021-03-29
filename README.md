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
