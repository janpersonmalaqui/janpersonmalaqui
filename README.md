#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define STRINGMAX 10

struct DateTime {
    int day;
    char month[STRINGMAX + 1];
    int year;
    int hour;
    int minute;
};

// Function 1: Read from file in day,month,year,hour,minute format
int readDataFile1(const char* name, struct DateTime records[], int maxCapacity) {
    FILE *file = fopen(name, "r");
    if (file == NULL) {
        return 0;
    }

    int count = 0;
    while (count < maxCapacity && fscanf(file, "%d,%s,%d,%d,%d\n", 
            &records[count].day, records[count].month, &records[count].year, 
            &records[count].hour, &records[count].minute) == 5) {
        count++;
    }

    fclose(file);  // Make sure to close the file
    return count;
}

// Function 2: Read from file in hour:minute am/pm;dd-Mon-yyyy format
int readDataFile2(const char* name, struct DateTime records[], int maxCapacity) {
    FILE *file = fopen(name, "r");
    if (file == NULL) {
        return 0;
    }

    int count = 0;
    char timeStr[20], dateStr[20];
    while (count < maxCapacity && fscanf(file, "%19[^;];%19[^\n]\n", timeStr, dateStr) == 2) {
        int hour, minute;
        char ampm[3], month[4];

        // Parse time
        if (sscanf(timeStr, "%d:%d %2s", &hour, &minute, ampm) == 3) {
            if (strcmp(ampm, "pm") == 0 && hour != 12) {
                hour += 12;  // Convert pm hour
            }
            if (strcmp(ampm, "am") == 0 && hour == 12) {
                hour = 0;  // Convert 12am to 00
            }
        }

        // Parse date
        if (sscanf(dateStr, "%d-%3s-%d", &records[count].day, month, &records[count].year) == 3) {
            strncpy(records[count].month, month, STRINGMAX);
            records[count].hour = hour;
            records[count].minute = minute;
        }

        count++;
    }

    fclose(file);  // Make sure to close the file
    return count;
}

// Function 3: Count occurrences of a specific month in the DateTime array
int numMonths(const char monthName[], const struct DateTime dates[], int size) {
    int count = 0;
    for (int i = 0; i < size; i++) {
        if (strcmp(dates[i].month, monthName) == 0) {
            count++;
        }
    }
    return count;
}

int main() {
    struct DateTime records[100];
    int numRecords;

    // Read data from the first file (comma-separated format)
    numRecords = readDataFile1("data1.txt", records, 100);
    printf("Records read from data1.txt: %d\n", numRecords);

    // Read data from the second file (semicolon-separated format)
    numRecords = readDataFile2("data2.txt", records, 100);
    printf("Records read from data2.txt: %d\n", numRecords);

    // Count records for a specific month (e.g., "Jan")
    int monthCount = numMonths("Jan", records, numRecords);
    printf("Number of records in January: %d\n", monthCount);

    return 0;
}
