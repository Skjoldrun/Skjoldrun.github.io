---
layout: page
title: C# - Check if in time scope
parent: C#
---

# Check if within time scope

It can come in handy to check if the execution time is within a time scope. A possible usecase could be that you would want to only send an alert mail within the working hours.

You could use the following code to achieve this:

```csharp
/// <summary>
/// Checks if the given time is within given working hours (Format: HH:mm).
/// </summary>
/// <param name="dateTimeNow">time to check</param>
/// <param name="periodStart">begin of working hours</param>
/// <param name="periodEnd">end of working hours</param>
/// <returns>true if within periodStart and periodEnd, else false</returns>
public bool CheckIfIsWithinWorkingHours(DateTime dateTimeNow, string periodStart, string periodEnd)
{
    var timeOfDayNow = dateTimeNow.TimeOfDay;
    if ((timeOfDayNow > DateTime.ParseExact(periodStart, "HH:mm", CultureInfo.InvariantCulture).TimeOfDay)
        && (timeOfDayNow < DateTime.ParseExact(periodEnd, "HH:mm", CultureInfo.InvariantCulture).TimeOfDay))
    {
        _logger.LogDebug("Checked time is within working hours: {timeOfDayNow}", timeOfDayNow);
        return true;
    }
    else
    {
        _logger.LogDebug("Checked time is not within working hours: {timeOfDayNow}", timeOfDayNow);
        return false;
    }
}
```

## Test

This can be testet with Memberdata of XUnit Tests:

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