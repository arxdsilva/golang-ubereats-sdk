# golang-ubereats-sdk
A golang ubereats sdk 

![Actions on main](https://github.com/arxdsilva/golang-ubereats-sdk/actions/workflows/test.yml/badge.svg?branch=main)
[![Coverage Status](https://coveralls.io/repos/github/arxdsilva/golang-ubereats-sdk/badge.svg?branch=main)](https://coveralls.io/github/arxdsilva/golang-ubereats-sdk?branch=main)
[![Go Report Card](https://goreportcard.com/badge/github.com/arxdsilva/golang-ubereats-sdk)](https://goreportcard.com/report/github.com/arxdsilva/golang-ubereats-sdk)
[![LICENSE](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)

## Usage

> WIP


```go

package main

import (
    sdk "github.com/arxdsilva/golang-ubereats-sdk/container"
)

func main() {
    var clientID, clientSecret, user, password string
    clientID = os.GetEnv("CLIENT_ID")
    clientSecret = os.GetEnv("CLIENT_SECRET")
    // START new SDK instance
    container := sdk.New(0, time.Minute)
    container.GetHttpAdapter()
    // Alocate services
    container.GetAuthenticationService(clientID, clientSecret)
    container.GetMerchantService()
    container.GetCatalogService()
    container.GetEventsService()
    container.GetOrdersService()
    user = os.GetEnv("USER")
    password = os.GetEnv("PASSWORD")
    creds, err := container.AuthService.Authenticate(user,password)
    if err != nil { 
        log.Fatal(err)
    }
    merchants, err := container.MerchantService.ListAll()
    if err != nil { 
        log.Fatal(err)
    }
    events, err := container.EventsService.Poll()
    if err != nil {
        log.Fatal(err)
    }
    var newOrdersDetails []orders.OrderDetails
	for _, event := range events {
        err = container.EventsService.Acknowledge(event)
        if err != nil {
            fmt.Println("err: ", err)
            continue
        }
        // avoid non new orders
        if event.Code != "PLACED" {
            continue
        }
        details, err := container.OrdersService.GetDetails(event.ID)
        if err != nil {
            fmt.Println("err: ", err)
            continue
        }
        newOrdersDetails = append(newOrdersDetails, details)
	}
	for _, order := range newOrdersDetails {
        // change order status
        err = container.OrdersService.SetIntegrateStatus(order.ID)
        if err != nil {
            fmt.Println("err: ", err)
            continue
        }
        // change other statuses
    }
    fmt.Printf("new orders: %+v\n", newOrdersDetails)
}
```