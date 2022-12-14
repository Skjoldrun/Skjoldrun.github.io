---
layout: page
title: C# - XUnit Test and memberData
parent: C#
---

# Use complex parameters with XUnit Tests and MemberData

Sometimes you need more complex parameter data to test in XUnit tests Theories and InlineData. For this scenario you can use MemberData with objects conatining the more complex parameters as shown below.


## Method to test

The method should check, if a given time is within a timespan between given `periodStart` and `periodEnd` parameters.

```csharp
public bool CheckIfIsWithinWorkingHours(DateTime dateTimeNow, string periodStart, string periodEnd)
{
    var timeOfDayNow = dateTimeNow.TimeOfDay;
    if ((timeOfDayNow > DateTime.ParseExact(periodStart, "HH:mm", CultureInfo.InvariantCulture).TimeOfDay)
        && (timeOfDayNow < DateTime.ParseExact(periodEnd, "HH:mm", CultureInfo.InvariantCulture).TimeOfDay))
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

## Test with MemberData

The upper Method can be tested with a XUnit Theory, but InlineData cannot handle DateTime objects besides the other wanted parameters. Therefore we can use the call for `[MemberData(nameof(ParamData))]` and the definition of a static Method named `ParamData`, wich returns objects with the needed parameters: 

```csharp
public class BeckhoffPulseCheckerTest
    {
        [Theory]
        [MemberData(nameof(ParamData))]
        public void CheckIfIsWithinWorkingHoursExpectTrueTest(DateTime dateTime, string periodStart, string periodEnd, bool expected)
        {
            // Arrange
            var beckhoffPulseChecker = new BeckhoffPulseChecker();

            // Act
            bool result = beckhoffPulseChecker.CheckIfIsWithinWorkingHours(dateTime, periodStart, periodEnd);

            // Assert
            Assert.Equal(expected, result);
        }

        public static IEnumerable<object[]> ParamData()
        {
            yield return new object[] { new DateTime(2022, 07, 07, 08, 00, 00), "07:00", "16:00", true };
            yield return new object[] { new DateTime(2022, 07, 07, 09, 00, 00), "07:00", "16:00", true };
            yield return new object[] { new DateTime(2022, 07, 07, 10, 00, 00), "07:00", "16:00", true };
            yield return new object[] { new DateTime(2022, 07, 07, 15, 00, 00), "07:00", "16:00", true };
            yield return new object[] { new DateTime(2022, 07, 07, 16, 01, 00), "07:00", "16:00", false };
            yield return new object[] { new DateTime(2022, 07, 07, 18, 00, 00), "07:00", "16:00", false };
            yield return new object[] { new DateTime(2022, 07, 07, 00, 00, 00), "07:00", "16:00", false };
            yield return new object[] { new DateTime(2022, 07, 07, 06, 00, 00), "07:00", "16:00", false };
        }
    }
```


You can mix datatypes and make really complex parameter combinations and easily call the testmethod with them.

[![Testmethod with parameters](/assets/images/coding/csharp/XUnit-test-memberData/testmethod-parameters.png)](/assets/images/coding/csharp/XUnit-test-memberData/testmethod-parameters.png)
    