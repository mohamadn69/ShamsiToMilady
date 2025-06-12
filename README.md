#
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
create FUNCTION [dbo].[ShamsiToMiladi](@ShamsiDate NVARCHAR(10))
RETURNS DATE
AS
BEGIN
    DECLARE @sy INT, @sm INT, @sd INT;
    DECLARE @isLeapYear BIT;
    DECLARE @baseDate DATE;
    DECLARE @days INT;
    DECLARE @MiladiDate DATE;

    -- استخراج سال، ماه، روز از رشته ورودی
    SET @sy = TRY_CAST(SUBSTRING(@ShamsiDate, 1, 4) AS INT);
    SET @sm = TRY_CAST(SUBSTRING(@ShamsiDate, 6, 2) AS INT);
    SET @sd = TRY_CAST(SUBSTRING(@ShamsiDate, 9, 2) AS INT);

    -- اعتبارسنجی اولیه
    IF @sy IS NULL OR @sm NOT BETWEEN 1 AND 12 OR @sd NOT BETWEEN 1 AND 31
        RETURN NULL;

    SET @isLeapYear = dbo.IsShamsiLeapYear_Exact(@sy);

    -- اسفند ۳۰ فقط در سال کبیسه معتبر است
    IF @sm = 12 AND @sd = 30 AND @isLeapYear = 0
        RETURN NULL;

    -- تعیین نوروز (۲۰ یا ۲۱ مارچ)
    SET @baseDate = DATEFROMPARTS(@sy + 621, 3, CASE WHEN @isLeapYear = 1 THEN 20 ELSE 21 END);

    -- محاسبه تعداد روزهای گذشته از نوروز
    IF @sm <= 6
        SET @days = (@sm - 1) * 31 + @sd - 1;
    ELSE
        SET @days = 6 * 31 + (@sm - 7) * 30 + @sd - 1;

    -- جمع با تاریخ نوروز
    SET @MiladiDate = DATEADD(DAY, @days, @baseDate);

    RETURN @MiladiDate;
END;
