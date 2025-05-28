# How-many-Reports

Here you can see how many report I maintain and calculate on. Each tab at the top provides a sub group underneith. There are filters for everything a User would need.

![image](https://github.com/user-attachments/assets/3a2fa465-0683-4f17-b30a-17feb0fe9271)


Here is a live pipeline from SQL to display in Power Bi and publish it. It shows how many holidays staff have booked. Clients prefer to see it in many formats so have included a table, a bar graph that also includes how many sick days they taken in same time span. 
For companies that have varied amount of holidays per staff I have included a pertcentage bar chart which shows orange where there is some holiday remaining. Note it highlighted Payroll number 22 as having more holiday 
than their yearly allowance. I have designed this with drill down and the bar chart will change to give you more details.

![image](https://github.com/user-attachments/assets/e0607c91-ab08-4b87-b983-fdb482ea50a8)


Here is another report design.
Holidays with Drill Down and slicer to choose date ranges

 ![image](https://github.com/user-attachments/assets/a7a227d9-dca4-4077-bbde-a32ec904e3ba)


Auto Adjust Drill Down to fit Holidays dates, showing Staff member Name and details of work. Note I am not keen on the length of each day and value 1, but does return a break down of each person should it be needed. 

![image](https://github.com/user-attachments/assets/b77e8fa8-379e-414a-a47a-3f626b92e3ab)


My next project is to Python this data from SQL and provide reports for web designers. The aim is to produce every report in the 1st picture and make it accessable globally.
Clients will log into the secure webpage and have a selection of reports on the cloud and do not did to visit the office to retreive them.


In order to get this data I can use a value stored in the database, but this relies on other to keep this correct. I have a script that with count the full and half day or how many hours have been booked as a Holiday and then the same for Sick.


![image](https://github.com/user-attachments/assets/c23889a3-faf5-4298-bcfd-4410c773e33e)


Here is the script from the picture and have many others


DROP FUNCTION IF EXISTS [dbo].[ConvertAnyTime]
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
Create function [dbo].[ConvertAnyTime]
(
    @time decimal(28,3)  
)
returns varchar(20)
as
begin
    declare @seconds decimal(18,3), @minutes int, @hours int,@unit varchar(20) = 'ss';
			if(@unit = 'hour' or @unit = 'hh' )
				set @seconds = @time * 60 * 60;
			else if(@unit = 'minute' or @unit = 'mi' or @unit = 'n')
				set @seconds = @time * 60;
			else if(@unit = 'second' or @unit = 'ss' or @unit = 's')
				set @seconds = @time;
			else set @seconds = 0; -- unknown time units

			--set @hours = convert(int, @seconds /60 / 60);
			--set @minutes = convert(int, (@seconds / 60) - (@hours * 60 ));
			--set @seconds = @seconds % 60;
	
if coalesce(@time,0)>=0
			set @hours = convert(int, @seconds /60 / 60)
if coalesce(@time,0)>=0
			set @minutes = CONVERT(int, (@seconds / 60) - (@hours * 60))
if coalesce(@time,0)>=0
			set @seconds = @seconds % 60;

if coalesce(@time,1)<0	
			set @hours = convert(int, @seconds /60 / 60);
if coalesce(@time,1)<0	
			set @minutes =CONVERT(int, (@seconds / 60) - (@hours * 60))-CONVERT(int, (@seconds / 60) - (@hours * 60))-CONVERT(int, (@seconds / 60) - (@hours * 60))
if coalesce(@time,1)<0	
			set @seconds = @seconds % 60;
	

    return 
        convert(varchar(9), convert(int, @hours)) + ':' +
        right('00' + convert(varchar(2), convert(int, @minutes)), 2) 
		--+ ':' +right('00' + convert(varchar(6), @seconds), 6)	
end
GO

With CTE as(
SELECT 0 Empref,0 CardNo,'' 'Forenames','' 'Surname',0 'Coderef','' 'GroupName','' 'CodeName','' 'WDay','' 'Date','' 'HolType',
    MAX(CASE WHEN rateno = 0 THEN ratesname END) AS Rate0,
    MAX(CASE WHEN rateno = 1 THEN ratesname END) AS Rate1,
    MAX(CASE WHEN rateno = 2 THEN ratesname END) AS Rate2,
    MAX(CASE WHEN rateno = 3 THEN ratesname END) AS Rate3,
    MAX(CASE WHEN rateno = 4 THEN ratesname END) AS Rate4,
    MAX(CASE WHEN rateno = 5 THEN ratesname END) AS Rate5,
    MAX(CASE WHEN rateno = 6 THEN ratesname END) AS Rate6,
    MAX(CASE WHEN rateno = 7 THEN ratesname END) AS Rate7,
	MAX(CASE WHEN rateno = 8 THEN ratesname END) AS Rate8,
    MAX(CASE WHEN rateno = 9 THEN ratesname END) AS Rate9,
    MAX(CASE WHEN rateno = 10 THEN ratesname END) AS Rate10,
	'' HolUnits,
	0 Total,
	0 RunningDays,
	0 TotalDays,
	0 RunningHrs,
	0 TotalHrs,
	0 PercentUsed,
	'' Absdate
FROM ratesnames

Union ALL
 select E.empref,
		E.cardno,
		E.forenames,
		E.surname,
		b.coderef,
		A.groupname,
		Ab.codename,
		datename (dw,b.absdate) as WDay,
		CONVERT(VARCHAR(8), b.absdate, 3) AS [DD/MM/YY],
		case when Abstype=1 then 'FullDay' when Abstype=2 then '1stHalf' when Abstype=3 then '2ndHalf' when Abstype = 5 then 'Part Day' end as HolType,
		--CONVERT(varchar, DATEADD(ms, B.Rate1 * 1000, 0), 114) as BasicRate,
		dbo.fn_ConvertIntToTimeStr(b.rate0) AS Rate0,
		dbo.fn_ConvertIntToTimeStr(b.rate1) AS Rate1,
		dbo.fn_ConvertIntToTimeStr(b.rate2) AS Rate2,
		dbo.fn_ConvertIntToTimeStr(b.Rate3) AS Rate3,
		dbo.fn_ConvertIntToTimeStr(b.Rate4) AS Rate4,
		dbo.fn_ConvertIntToTimeStr(b.Rate5) AS Rate5,
		dbo.fn_ConvertIntToTimeStr(b.Rate6) AS Rate6,
		dbo.fn_ConvertIntToTimeStr(b.rate7) AS Rate7,
		dbo.fn_ConvertIntToTimeStr(b.rate8) AS Rate8,
		dbo.fn_ConvertIntToTimeStr(b.rate9) AS Rate9,
		dbo.fn_ConvertIntToTimeStr(b.rate10) AS Rate10,
		Case when ED.enttype = 0 then 'Days =' else 'Hours = ' End ,
		coalesce(NULLIF(ED.ContractedHoliday+ED.Entamount,0),0) Total,
		SUM(case when B.Abstype=1 then 1.0 when b.Abstype=2 then 0.5 when B.Abstype=3 then 0.5 end) OVER (PARTITION By B.empref order by b.absdate) as RunningDays,
		SUM(case when B.Abstype=1 then 1.0 when b.Abstype=2 then 0.5 when B.Abstype=3 then 0.5 end) OVER (PARTITION By B.empref) as TotalNoDays,
		sum (B.rate1+ B.rate2 +B.rate3 +B.rate4 +B.rate5 +B.rate6 +B.rate7 +B.rate8 +B.rate9 +B.rate10) over (partition by B.Empref order by B.absdate,b.coderef) as DayHoursRunning,
		sum (B.rate1+ B.rate2 +B.rate3 +B.rate4 +B.rate5 +B.rate6 +B.rate7 +B.rate8 +B.rate9 +B.rate10) over (partition by B.Empref ) as DayHours,
		cast(case 
				when coalesce(NULLIF(ED.ContractedHoliday+ED.Entamount,0),0) = 0 then 100
				when ED.enttype = 0 then SUM(case when B.Abstype=1 then 1.0 when b.Abstype=2 then 0.5 when B.Abstype=3 then 0.5 end) OVER (PARTITION By B.empref) /(coalesce(NULLIF(ED.ContractedHoliday+ED.Entamount,0),100))*100
				else sum (B.rate1+ B.rate2 +B.rate3 +B.rate4 +B.rate5 +B.rate6 +B.rate7 +B.rate8 +B.rate9 +B.rate10
		) over (partition by B.Empref )*100 /(coalesce(NULLIF(ED.ContractedHoliday*3600+ED.Entamount*3600,0),100)) END as Decimal (10,1)) as PercentUsed,
		absdate
		--ed.entamount
		from vwabsences B
		join Empdetails E on E.empref = B.empref
		left join abscodes Ab on Ab.coderef = b.coderef
		left join absgroups A on A.groupref = B.groupref
		left join entitlementdefault ED on ED.groupref = E.HolEntGroupRef
		where
		Ab.groupref = 1 and
		-- surname = 'norman' 
		b.Empref in(184,174) and 
		--e.payrollno = '507' and
		absdate between ed.entstartdate and ed.entenddate
	--select * from entitlementdefault
)	
Select 
Empref,
CardNo,
Forenames,
Surname,
Coderef,
GroupName,
CodeName,
WDay,
Date,
HolType,
coalesce(rate0,'') as Rate0,
coalesce(rate1,'') as Rate1,
coalesce(rate2,'') as Rate2,
coalesce(rate3,'') as Rate3,
coalesce(rate4,'') as Rate4,
coalesce(rate5,'') as Rate5,
coalesce(rate6,'') as Rate6,
coalesce(rate7,'') as Rate7,
coalesce(rate8,'') as Rate8,
coalesce(rate9,'') as Rate9,
coalesce(rate10,'') as Rate10,
HolUnits,
Total,
RunningDays,
TotalDays,
dbo.ConvertAnyTime(RunningHrs) as RunningHrs,
dbo.ConvertAnyTime(TotalHrs) as TotalHrs,
cast(percentused as Varchar (6)) + '%' as PercentUsed
from CTE

order by forenames,surname,Absdate
Drop function [dbo].[ConvertAnyTime]
