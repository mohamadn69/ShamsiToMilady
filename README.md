







create FUNCTION [dbo].[IsShamsiLeapYear_Exact](@sy INT)
RETURNS BIT
AS
BEGIN
    DECLARE @mod INT = @sy % 33;
    IF (@sy BETWEEN 1244 AND 1342 AND @mod IN (1, 5, 9, 13, 17, 21, 26, 30))
        RETURN 1;
    ELSE IF (@sy BETWEEN 1343 AND 1472 AND @mod IN (1, 5, 9, 13, 17, 22, 26, 30))
        RETURN 1;
    RETURN 0;
END;
