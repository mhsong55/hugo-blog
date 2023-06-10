---
title: "Azure Application Gateway"
date: 2022-04-30
draft: true
author: "MyungHoon Song"
description: "Azure의 L7 load balancer"
hideSummary: false
tags:
- Azure
---

# 포스트 테스트

시간대 좀 어케 해봐라

```pwsh
function Request-HolidayInfo {
    param (
        [string]$yearMonth
    )
    $year = $yearMonth.Split("-")[0]
    $month = $yearMonth.Split("-")[1]
    $key = "wM5w4H0Cey242vo7gB%2BCdii5nIhNsaxgIrZV9ZW15OZkiDZ7h%2BQ5G%2B8zXYrS8h6Baf959UXUvcDcZNSsN5YuSg%3D%3D"
    $endpoint = "http://apis.data.go.kr/B090041/openapi/service/SpcdeInfoService"
    $uri = "$endpoint/getHoliDeInfo?ServiceKey=$key&solYear=$year&solMonth=$month&_type=json"
    $response = Invoke-RestMethod -Uri $uri -Method:Get
    foreach ($item in $response.response.body.items.item) {
        Write-Output $item
    }
    return $response.response.body.items
}
function IsHoliday {
    param (
        [datetime] $Date,
        [Object[]] $HolidayArray
    )
    foreach ($item in $HolidayArray) {
        if ($Date.ToString("yyyyMMdd") -eq [string]$item.locdate) {
            Write-Host "$($Date.ToString("yyyy-MM-dd"))는 공휴일($($item.dateName)) 입니다."
            return $true
        }
    }
    if (($Date.DayOfWeek -eq [DayOfWeek]::Saturday) -or ($Date.DayOfWeek -eq [DayOfWeek]::Sunday)) { 
        return $true
    }
    return $false
}
function Get-WorkingDayExcel {
    param (
        [datetime] $Today,
        [string] $FilePath,
        [string] $FileName
    )
    $datePrefix = $Today.ToString("yyyy-MM")
    $dateList = [System.Collections.ArrayList]::new()
    $nextMonth = $Today.addMonths(1).Month
    $nextDay = $Today.AddDays(1)
    $count = 1
    $holiday = Request-HolidayInfo -yearMonth $datePrefix
    while ($Today.Month -ne $nextMonth) {
        Write-Output "count = $count, Today = $Today, NextDay = $nextDay"
        $dateRecord = "" | Select-Object date, dayOfWeek, isHoliday, isDirectGo, isVacation
        $dateRecord.date = $Today.ToString("yyyy-MM-dd")
        $dateRecord.dayOfWeek = $Today.DayOfWeek
        $dateRecord.isHoliday = IsHoliday -Date $Today -HolidayArray $holiday
        $dateRecord.isDirectGo = $false
        $dateRecord.isVacation = $false
        [void]$dateList.Add($dateRecord)
        $Today = $Today.AddDays(1)
        $nextDay = $Today.AddDays(1)
        $count++
    }
    $dateList | Export-Csv -Path "$FilePath/$FileName.csv" -NoTypeInformation -Encoding:utf8BOM
    Import-Csv "$FilePath/$FileName.csv" | Export-Excel "$FilePath/$FileName.xlsx"
    Remove-Item "$FilePath/$FileName.csv"
}
# $today = (Get-Date).AddDays(1)
$today = (Get-Date).AddDays(-3)
$filePrefix = $today.ToString("yyyy-MM")
$filePath = ".\"
$fileName = "${filePrefix}_WorkDayTime"
if ((Get-Item -Path "$filePath*").Name.Contains("$fileName.xlsx")) {
    Write-Host "$fileName.xlsx 파일이 존재하여 작업을 중단합니다."
    break
} else {
    Write-Host "$fileName.xlsx 파일이 존재하지 않습니다. 작업을 진행합니다."
    Get-WorkingDayExcel -Today $today -FilePath $filePath -FileName $fileName
}
```




```java
package datastructure.list;

import java.sql.Array;

public class ArrayList<E> implements List<E> {
    private E item[];
    private int numItems;
    private static final int DEFAULT_CAPACITY = 64;
    public ArrayList() {
        this.item = (E[]) new Object[DEFAULT_CAPACITY];
        this.numItems = 0;
    }
    public ArrayList(int n) {
        this.item = (E[]) new Object[n];
        this.numItems = 0;
    }

    // ArrayList의 index 번째 자리에 원소 e 삽입하기
    @Override
    public void add(int index, E e) {
        if(numItems >= item.length || index < 0 || index > numItems ) {
            throw new ArrayIndexOutOfBoundsException();
        } else {
            for (int i = numItems - 1; i >= index; i--) {
                item[i + 1] = item[i]; // right shift
            }
            item[index] = e;
            numItems++;
        }
    }

    // 배열 리스트의 맨 뒤에 원소 추가하기
    @Override
    public void append(E e) {
        if (numItems >= item.length) {
            throw new ArrayIndexOutOfBoundsException();
        } else {
            item[numItems++] = e;
        }
    }

    // ArrayList의 index 번째 원소 삭제
    @Override
    public E remove(int index) {
        if (isEmpty() || index < 0 || index > numItems - 1) {
            return null;
        } else {
            E tmp = item[index];
            for (int i = index; i <= numItems - 2; i++) {
                item[i] = item[i+1]; // Left shift
            }
            numItems--;
            return tmp;
        }
    }

    // ArrayList에서 원소 e 삭제하기
    @Override
    public boolean removeItem(E e) {
        int index = 0;
        while (index < numItems && ((Comparable)item[index]).compareTo(e) != 0) { // 원소 e 찾기
            index++;
        }
        if (index == numItems){             // 원소 e 없음
            return false;
        } else {
            for (int i = index; i <= numItems - 2; i++) {
                item[i] = item[i+1];        // Left shift
            }
            numItems--;
            return true;
        }
    }

    // 리스트의 index 번째 원소 알려주기
    @Override
    public E get(int index) {
        if (index >= 0 && index <= numItems - 1) {
            return item[index];
        } else {
            return null;
        }
    }

    // ArrayList의 index 번째 원소를 e로 대체하기
    @Override
    public void set(int index, E e) {
        if (index >= 0 && index <= numItems - 1) {
            item[index] = e;
        } else {
            /* 에러 처리 */
        }
    }

    // 원소 e가 Array List의 몇 번째 원소인지 알려주기
    private final int NOT_FOUND = -12345;
    @Override
    public int indexOf(E e) {
        int i = 0;
        for (i = 0; i < numItems; i++) {
            if(((Comparable)item[i]).compareTo(e) == 0) {
                return i;
            }
        }
        return NOT_FOUND;
    }

    // 리스트의 총 원소 수 알려주기
    @Override
    public int len() {
        return numItems;
    }

    // List가 비었는지 알려주기
    @Override
    public boolean isEmpty() {
        return numItems == 0;
    }

    // list 청소하기
    @Override
    public void clear() {
        item = (E[]) new Object[item.length];
        numItems = 0;
    }
}

```