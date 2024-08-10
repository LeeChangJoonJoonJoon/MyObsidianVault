

2023-11-06

----
#Json #Cpp #Unreal 

## 개요
다음 JSON 파일을 http://52.79.98.84/entities 에 요청하여 받은 다음, 파싱해야 하는 상황이다.
```Json
[
  {
    "id": "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101D",
    "type": "DirectFiredAbsorptionChillerHeater",
    "equipmentName": {
      "type": "Property",
      "value": "흡수식 냉온수기 D"
    },
    "connectTo": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005",
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "category": {
      "type": "Property",
      "value": "heatSource"
    },
    "subCategory": {
      "type": "Property",
      "value": "directFiredAbsorptionChiller"
    },
    "usage": {
      "type": "Property",
      "value": "heatCool"
    },
    "equipmentNumber": {
      "type": "Property",
      "value": "R-101D"
    },
    "locatedIn": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Room:R002"
      ]
    },
    "installationLocation": {
      "type": "Property",
      "value": "urn:ngsi-ld:Room:R002"
    },
    "lowerConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005"
      ]
    },
    "upperConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "lineGroup": {
      "type": "Property",
      "value": {
        "coolingWaterSupplyLineGroup": {
          "name": "냉각수 공급관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101D1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101D2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101D3",
            "urn:ngsi-ld:Pipe:PPS-CLW-00010",
            "urn:ngsi-ld:Pipe:PPS-CLW-00011",
            "urn:ngsi-ld:Pipe:PPS-CLW-00012",
            "urn:ngsi-ld:Pipe:PPS-CLW-00013",
            "urn:ngsi-ld:Pipe:PPS-CLW-00015",
            "urn:ngsi-ld:Pipe:PPS-CLW-00016",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101D"
          ]
        },
        "coolingWaterReturnLineGroup": {
          "name": "냉각수 환수관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101D1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101D2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101D3",
            "urn:ngsi-ld:Pipe:PPR-CLW-00010",
            "urn:ngsi-ld:Pipe:PPR-CLW-00011",
            "urn:ngsi-ld:Pipe:PPR-CLW-00012",
            "urn:ngsi-ld:Pipe:PPR-CLW-00013",
            "urn:ngsi-ld:Pipe:PPR-CLW-00015",
            "urn:ngsi-ld:Pipe:PPR-CLW-00017",
            "urn:ngsi-ld:Pipe:PPR-CLW-00018",
            "urn:ngsi-ld:Pipe:PPR-CLW-00019",
            "urn:ngsi-ld:Pipe:PPR-CLW-00020",
            "urn:ngsi-ld:Pump:EQM-PUM-0111A",
            "urn:ngsi-ld:Pump:EQM-PUM-0111B",
            "urn:ngsi-ld:Pump:EQM-PUM-0111C",
            "urn:ngsi-ld:Pump:EQM-PUM-0111D",
            "urn:ngsi-ld:Pipe:PPR-CLW-00021",
            "urn:ngsi-ld:Pipe:PPR-CLW-00025",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101D"
          ]
        },
        "chilledWaterSupplyLineGroup": {
          "name": "냉수 공급관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00028",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPS-CDW-00016",
            "urn:ngsi-ld:Pipe:PPS-CDW-00017",
            "urn:ngsi-ld:Pipe:PPS-CDW-00015",
            "urn:ngsi-ld:Pipe:PPS-CDW-00024",
            "urn:ngsi-ld:Pipe:PPS-CDW-00012",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPS-CDW-00029",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00019",
            "urn:ngsi-ld:Pipe:PPS-CDW-00020",
            "urn:ngsi-ld:Pipe:PPS-CDW-00018",
            "urn:ngsi-ld:Pipe:PPS-CDW-00025",
            "urn:ngsi-ld:Pipe:PPS-CDW-00026",
            "urn:ngsi-ld:Pipe:PPS-CDW-00013",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPS-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00022",
            "urn:ngsi-ld:Pipe:PPS-CDW-00023",
            "urn:ngsi-ld:Pipe:PPS-CDW-00021",
            "urn:ngsi-ld:Pipe:PPS-CDW-00027",
            "urn:ngsi-ld:Pipe:PPS-CDW-00014",
            "urn:ngsi-ld:Pipe:PPS-CDW-00011",
            "urn:ngsi-ld:Pump:EQM-PUM-0118A",
            "urn:ngsi-ld:Pump:EQM-PUM-0118B",
            "urn:ngsi-ld:Pump:EQM-PUM-0118C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00008",
            "urn:ngsi-ld:Pipe:PPS-CDW-00009",
            "urn:ngsi-ld:Pipe:PPS-CDW-00010",
            "urn:ngsi-ld:Pipe:PPS-CDW-00006",
            "urn:ngsi-ld:Pipe:PPS-CDW-00005",
            "urn:ngsi-ld:Pipe:PPS-CDW-00004",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101D"
          ]
        },
        "chilledWaterReturnLineGroup": {
          "name": "냉수 환수관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPR-CDW-00018",
            "urn:ngsi-ld:Pipe:PPR-CDW-00019",
            "urn:ngsi-ld:Pipe:PPR-CDW-00017",
            "urn:ngsi-ld:Pipe:PPR-CDW-00026",
            "urn:ngsi-ld:Pipe:PPR-CDW-00014",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPR-CDW-00031",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPR-CDW-00021",
            "urn:ngsi-ld:Pipe:PPR-CDW-00022",
            "urn:ngsi-ld:Pipe:PPR-CDW-00020",
            "urn:ngsi-ld:Pipe:PPR-CDW-00027",
            "urn:ngsi-ld:Pipe:PPR-CDW-00028",
            "urn:ngsi-ld:Pipe:PPR-CDW-00015",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPR-CDW-00032",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00024",
            "urn:ngsi-ld:Pipe:PPR-CDW-00025",
            "urn:ngsi-ld:Pipe:PPR-CDW-00023",
            "urn:ngsi-ld:Pipe:PPR-CDW-00029",
            "urn:ngsi-ld:Pipe:PPR-CDW-00016",
            "urn:ngsi-ld:Pipe:PPR-CDW-00001",
            "urn:ngsi-ld:Pipe:PPR-CDW-00002",
            "urn:ngsi-ld:Pipe:PPR-CDW-00003",
            "urn:ngsi-ld:Pump:EQM-PUM-0114A",
            "urn:ngsi-ld:Pump:EQM-PUM-0114B",
            "urn:ngsi-ld:Pump:EQM-PUM-0114C",
            "urn:ngsi-ld:Pump:EQM-PUM-0114D",
            "urn:ngsi-ld:Pipe:PPR-CDW-00004",
            "urn:ngsi-ld:Pipe:PPR-CDW-00005",
            "urn:ngsi-ld:Pipe:PPR-CDW-00006",
            "urn:ngsi-ld:Pipe:PPR-CDW-00007",
            "urn:ngsi-ld:Pipe:PPR-CDW-00009",
            "urn:ngsi-ld:Pipe:PPR-CDW-00010",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101D"
          ]
        },
        "gasSupplyLineGroup": {
          "name": "가스 공급관",
          "list": [
            "urn:ngsi-ld:Pipe:PPS-GAS-00001",
            "urn:ngsi-ld:Pipe:PPS-GAS-00002",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101D"
          ]
        }
      }
    },
    "ratedCoolingWaterFlowRate": {
      "type": "Property",
      "value": "10180"
    },
    "@context": "[http://172.16.28.221:53005/e8ight-context.jsonld](http://172.16.28.221:53005/e8ight-context.jsonld)"
  },
  {
    "id": "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101A",
    "type": "DirectFiredAbsorptionChillerHeater",
    "equipmentName": {
      "type": "Property",
      "value": "흡수식 냉온수기 A"
    },
    "connectTo": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005",
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "category": {
      "type": "Property",
      "value": "heatSource"
    },
    "subCategory": {
      "type": "Property",
      "value": "directFiredAbsorptionChiller"
    },
    "usage": {
      "type": "Property",
      "value": "heatCool"
    },
    "equipmentNumber": {
      "type": "Property",
      "value": "R-101A"
    },
    "locatedIn": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Room:R002"
      ]
    },
    "installationLocation": {
      "type": "Property",
      "value": "urn:ngsi-ld:Room:R002"
    },
    "lowerConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005"
      ]
    },
    "upperConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "lineGroup": {
      "type": "Property",
      "value": {
        "coolingWaterSupplyLineGroup": {
          "name": "냉각수 공급관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101A1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101A2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101A3",
            "urn:ngsi-ld:Pipe:PPS-CLW-00001",
            "urn:ngsi-ld:Pipe:PPS-CLW-00002",
            "urn:ngsi-ld:Pipe:PPS-CLW-00003",
            "urn:ngsi-ld:Pipe:PPS-CLW-00014",
            "urn:ngsi-ld:Pipe:PPS-CLW-00015",
            "urn:ngsi-ld:Pipe:PPS-CLW-00019",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101A"
          ]
        },
        "coolingWaterReturnLineGroup": {
          "name": "냉각수 환수관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101A1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101A2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101A3",
            "urn:ngsi-ld:Pipe:PPR-CLW-00001",
            "urn:ngsi-ld:Pipe:PPR-CLW-00002",
            "urn:ngsi-ld:Pipe:PPR-CLW-00003",
            "urn:ngsi-ld:Pipe:PPR-CLW-00014",
            "urn:ngsi-ld:Pipe:PPR-CLW-00015",
            "urn:ngsi-ld:Pipe:PPR-CLW-00017",
            "urn:ngsi-ld:Pipe:PPR-CLW-00018",
            "urn:ngsi-ld:Pipe:PPR-CLW-00019",
            "urn:ngsi-ld:Pipe:PPR-CLW-00020",
            "urn:ngsi-ld:Pump:EQM-PUM-0111A",
            "urn:ngsi-ld:Pump:EQM-PUM-0111B",
            "urn:ngsi-ld:Pump:EQM-PUM-0111C",
            "urn:ngsi-ld:Pump:EQM-PUM-0111D",
            "urn:ngsi-ld:Pipe:PPR-CLW-00021",
            "urn:ngsi-ld:Pipe:PPR-CLW-00022",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101A"
          ]
        },
        "chilledWaterSupplyLineGroup": {
          "name": "냉수 공급관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00028",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPS-CDW-00016",
            "urn:ngsi-ld:Pipe:PPS-CDW-00017",
            "urn:ngsi-ld:Pipe:PPS-CDW-00015",
            "urn:ngsi-ld:Pipe:PPS-CDW-00024",
            "urn:ngsi-ld:Pipe:PPS-CDW-00012",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPS-CDW-00029",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00019",
            "urn:ngsi-ld:Pipe:PPS-CDW-00020",
            "urn:ngsi-ld:Pipe:PPS-CDW-00018",
            "urn:ngsi-ld:Pipe:PPS-CDW-00025",
            "urn:ngsi-ld:Pipe:PPS-CDW-00026",
            "urn:ngsi-ld:Pipe:PPS-CDW-00013",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPS-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00022",
            "urn:ngsi-ld:Pipe:PPS-CDW-00023",
            "urn:ngsi-ld:Pipe:PPS-CDW-00021",
            "urn:ngsi-ld:Pipe:PPS-CDW-00027",
            "urn:ngsi-ld:Pipe:PPS-CDW-00014",
            "urn:ngsi-ld:Pipe:PPS-CDW-00011",
            "urn:ngsi-ld:Pump:EQM-PUM-0118A",
            "urn:ngsi-ld:Pump:EQM-PUM-0118B",
            "urn:ngsi-ld:Pump:EQM-PUM-0118C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00008",
            "urn:ngsi-ld:Pipe:PPS-CDW-00009",
            "urn:ngsi-ld:Pipe:PPS-CDW-00010",
            "urn:ngsi-ld:Pipe:PPS-CDW-00006",
            "urn:ngsi-ld:Pipe:PPS-CDW-00005",
            "urn:ngsi-ld:Pipe:PPS-CDW-00001",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101A"
          ]
        },
        "chilledWaterReturnLineGroup": {
          "name": "냉수 환수관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPR-CDW-00018",
            "urn:ngsi-ld:Pipe:PPR-CDW-00019",
            "urn:ngsi-ld:Pipe:PPR-CDW-00017",
            "urn:ngsi-ld:Pipe:PPR-CDW-00026",
            "urn:ngsi-ld:Pipe:PPR-CDW-00014",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPR-CDW-00031",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPR-CDW-00021",
            "urn:ngsi-ld:Pipe:PPR-CDW-00022",
            "urn:ngsi-ld:Pipe:PPR-CDW-00020",
            "urn:ngsi-ld:Pipe:PPR-CDW-00027",
            "urn:ngsi-ld:Pipe:PPR-CDW-00028",
            "urn:ngsi-ld:Pipe:PPR-CDW-00015",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPR-CDW-00032",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00024",
            "urn:ngsi-ld:Pipe:PPR-CDW-00025",
            "urn:ngsi-ld:Pipe:PPR-CDW-00023",
            "urn:ngsi-ld:Pipe:PPR-CDW-00029",
            "urn:ngsi-ld:Pipe:PPR-CDW-00016",
            "urn:ngsi-ld:Pipe:PPR-CDW-00001",
            "urn:ngsi-ld:Pipe:PPR-CDW-00002",
            "urn:ngsi-ld:Pipe:PPR-CDW-00003",
            "urn:ngsi-ld:Pump:EQM-PUM-0114A",
            "urn:ngsi-ld:Pump:EQM-PUM-0114B",
            "urn:ngsi-ld:Pump:EQM-PUM-0114C",
            "urn:ngsi-ld:Pump:EQM-PUM-0114D",
            "urn:ngsi-ld:Pipe:PPR-CDW-00004",
            "urn:ngsi-ld:Pipe:PPR-CDW-00005",
            "urn:ngsi-ld:Pipe:PPR-CDW-00006",
            "urn:ngsi-ld:Pipe:PPR-CDW-00007",
            "urn:ngsi-ld:Pipe:PPR-CDW-00009",
            "urn:ngsi-ld:Pipe:PPR-CDW-00013",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101A"
          ]
        },
        "gasSupplyLineGroup": {
          "name": "가스 공급관",
          "list": [
            "urn:ngsi-ld:Pipe:PPS-GAS-00001",
            "urn:ngsi-ld:Pipe:PPS-GAS-00005",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101A"
          ]
        }
      }
    },
    "ratedCoolingWaterFlowRate": {
      "type": "Property",
      "value": "10180"
    },
    "@context": "[http://172.16.28.221:53005/e8ight-context.jsonld](http://172.16.28.221:53005/e8ight-context.jsonld)"
  },
  {
    "id": "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101C",
    "type": "DirectFiredAbsorptionChillerHeater",
    "equipmentName": {
      "type": "Property",
      "value": "흡수식 냉온수기 C"
    },
    "connectTo": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005",
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "category": {
      "type": "Property",
      "value": "heatSource"
    },
    "subCategory": {
      "type": "Property",
      "value": "directFiredAbsorptionChiller"
    },
    "usage": {
      "type": "Property",
      "value": "heatCool"
    },
    "equipmentNumber": {
      "type": "Property",
      "value": "R-101C"
    },
    "locatedIn": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Room:R002"
      ]
    },
    "installationLocation": {
      "type": "Property",
      "value": "urn:ngsi-ld:Room:R002"
    },
    "lowerConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005"
      ]
    },
    "upperConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "lineGroup": {
      "type": "Property",
      "value": {
        "coolingWaterSupplyLineGroup": {
          "name": "냉각수 공급관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101C1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101C2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101C3",
            "urn:ngsi-ld:Pipe:PPS-CLW-00007",
            "urn:ngsi-ld:Pipe:PPS-CLW-00008",
            "urn:ngsi-ld:Pipe:PPS-CLW-00009",
            "urn:ngsi-ld:Pipe:PPS-CLW-00014",
            "urn:ngsi-ld:Pipe:PPS-CLW-00015",
            "urn:ngsi-ld:Pipe:PPS-CLW-00017",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101C"
          ]
        },
        "coolingWaterReturnLineGroup": {
          "name": "냉각수 환수관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101C1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101C2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101C3",
            "urn:ngsi-ld:Pipe:PPR-CLW-00007",
            "urn:ngsi-ld:Pipe:PPR-CLW-00008",
            "urn:ngsi-ld:Pipe:PPR-CLW-00009",
            "urn:ngsi-ld:Pipe:PPR-CLW-00014",
            "urn:ngsi-ld:Pipe:PPR-CLW-00015",
            "urn:ngsi-ld:Pipe:PPR-CLW-00017",
            "urn:ngsi-ld:Pipe:PPR-CLW-00018",
            "urn:ngsi-ld:Pipe:PPR-CLW-00019",
            "urn:ngsi-ld:Pipe:PPR-CLW-00020",
            "urn:ngsi-ld:Pump:EQM-PUM-0111A",
            "urn:ngsi-ld:Pump:EQM-PUM-0111B",
            "urn:ngsi-ld:Pump:EQM-PUM-0111C",
            "urn:ngsi-ld:Pump:EQM-PUM-0111D",
            "urn:ngsi-ld:Pipe:PPR-CLW-00021",
            "urn:ngsi-ld:Pipe:PPR-CLW-00024",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101C"
          ]
        },
        "chilledWaterSupplyLineGroup": {
          "name": "냉수 공급관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00028",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPS-CDW-00016",
            "urn:ngsi-ld:Pipe:PPS-CDW-00017",
            "urn:ngsi-ld:Pipe:PPS-CDW-00015",
            "urn:ngsi-ld:Pipe:PPS-CDW-00024",
            "urn:ngsi-ld:Pipe:PPS-CDW-00012",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPS-CDW-00029",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00019",
            "urn:ngsi-ld:Pipe:PPS-CDW-00020",
            "urn:ngsi-ld:Pipe:PPS-CDW-00018",
            "urn:ngsi-ld:Pipe:PPS-CDW-00025",
            "urn:ngsi-ld:Pipe:PPS-CDW-00026",
            "urn:ngsi-ld:Pipe:PPS-CDW-00013",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPS-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00022",
            "urn:ngsi-ld:Pipe:PPS-CDW-00023",
            "urn:ngsi-ld:Pipe:PPS-CDW-00021",
            "urn:ngsi-ld:Pipe:PPS-CDW-00027",
            "urn:ngsi-ld:Pipe:PPS-CDW-00014",
            "urn:ngsi-ld:Pipe:PPS-CDW-00011",
            "urn:ngsi-ld:Pump:EQM-PUM-0118A",
            "urn:ngsi-ld:Pump:EQM-PUM-0118B",
            "urn:ngsi-ld:Pump:EQM-PUM-0118C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00008",
            "urn:ngsi-ld:Pipe:PPS-CDW-00009",
            "urn:ngsi-ld:Pipe:PPS-CDW-00010",
            "urn:ngsi-ld:Pipe:PPS-CDW-00006",
            "urn:ngsi-ld:Pipe:PPS-CDW-00005",
            "urn:ngsi-ld:Pipe:PPS-CDW-00003",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101C"
          ]
        },
        "chilledWaterReturnLineGroup": {
          "name": "냉수 환수관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPR-CDW-00018",
            "urn:ngsi-ld:Pipe:PPR-CDW-00019",
            "urn:ngsi-ld:Pipe:PPR-CDW-00017",
            "urn:ngsi-ld:Pipe:PPR-CDW-00026",
            "urn:ngsi-ld:Pipe:PPR-CDW-00014",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPR-CDW-00031",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPR-CDW-00021",
            "urn:ngsi-ld:Pipe:PPR-CDW-00022",
            "urn:ngsi-ld:Pipe:PPR-CDW-00020",
            "urn:ngsi-ld:Pipe:PPR-CDW-00027",
            "urn:ngsi-ld:Pipe:PPR-CDW-00028",
            "urn:ngsi-ld:Pipe:PPR-CDW-00015",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPR-CDW-00032",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00024",
            "urn:ngsi-ld:Pipe:PPR-CDW-00025",
            "urn:ngsi-ld:Pipe:PPR-CDW-00023",
            "urn:ngsi-ld:Pipe:PPR-CDW-00029",
            "urn:ngsi-ld:Pipe:PPR-CDW-00016",
            "urn:ngsi-ld:Pipe:PPR-CDW-00001",
            "urn:ngsi-ld:Pipe:PPR-CDW-00002",
            "urn:ngsi-ld:Pipe:PPR-CDW-00003",
            "urn:ngsi-ld:Pump:EQM-PUM-0114A",
            "urn:ngsi-ld:Pump:EQM-PUM-0114B",
            "urn:ngsi-ld:Pump:EQM-PUM-0114C",
            "urn:ngsi-ld:Pump:EQM-PUM-0114D",
            "urn:ngsi-ld:Pipe:PPR-CDW-00004",
            "urn:ngsi-ld:Pipe:PPR-CDW-00005",
            "urn:ngsi-ld:Pipe:PPR-CDW-00006",
            "urn:ngsi-ld:Pipe:PPR-CDW-00007",
            "urn:ngsi-ld:Pipe:PPR-CDW-00009",
            "urn:ngsi-ld:Pipe:PPR-CDW-00011",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101C"
          ]
        },
        "gasSupplyLineGroup": {
          "name": "가스 공급관",
          "list": [
            "urn:ngsi-ld:Pipe:PPS-GAS-00001",
            "urn:ngsi-ld:Pipe:PPS-GAS-00003",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101C"
          ]
        }
      }
    },
    "ratedCoolingWaterFlowRate": {
      "type": "Property",
      "value": "10180"
    },
    "@context": "[http://172.16.28.221:53005/e8ight-context.jsonld](http://172.16.28.221:53005/e8ight-context.jsonld)"
  },
  {
    "id": "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101B",
    "type": "DirectFiredAbsorptionChillerHeater",
    "equipmentName": {
      "type": "Property",
      "value": "흡수식 냉온수기 B"
    },
    "connectTo": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005",
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "category": {
      "type": "Property",
      "value": "heatSource"
    },
    "subCategory": {
      "type": "Property",
      "value": "directFiredAbsorptionChiller"
    },
    "usage": {
      "type": "Property",
      "value": "heatCool"
    },
    "equipmentNumber": {
      "type": "Property",
      "value": "R-101B"
    },
    "locatedIn": {
      "type": "Relationship",
      "object": [
        "urn:ngsi-ld:Room:R002"
      ]
    },
    "installationLocation": {
      "type": "Property",
      "value": "urn:ngsi-ld:Room:R002"
    },
    "lowerConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0114A",
        "urn:ngsi-ld:Pipe:PPR-CDW-00002",
        "urn:ngsi-ld:Pipe:PPS-CDW-00005"
      ]
    },
    "upperConnectedEquipments": {
      "type": "Property",
      "value": [
        "urn:ngsi-ld:Pump:EQM-PUM-0111A"
      ]
    },
    "lineGroup": {
      "type": "Property",
      "value": {
        "coolingWaterSupplyLineGroup": {
          "name": "냉각수 공급관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101B1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101B2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101B3",
            "urn:ngsi-ld:Pipe:PPS-CLW-00004",
            "urn:ngsi-ld:Pipe:PPS-CLW-00005",
            "urn:ngsi-ld:Pipe:PPS-CLW-00006",
            "urn:ngsi-ld:Pipe:PPS-CLW-00013",
            "urn:ngsi-ld:Pipe:PPS-CLW-00015",
            "urn:ngsi-ld:Pipe:PPS-CLW-00018",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101B"
          ]
        },
        "coolingWaterReturnLineGroup": {
          "name": "냉각수 환수관",
          "list": [
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101B1",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101B2",
            "urn:ngsi-ld:CoolingTower:EQM-CLT-101B3",
            "urn:ngsi-ld:Pipe:PPR-CLW-00004",
            "urn:ngsi-ld:Pipe:PPR-CLW-00005",
            "urn:ngsi-ld:Pipe:PPR-CLW-00006",
            "urn:ngsi-ld:Pipe:PPR-CLW-00013",
            "urn:ngsi-ld:Pipe:PPR-CLW-00015",
            "urn:ngsi-ld:Pipe:PPR-CLW-00017",
            "urn:ngsi-ld:Pipe:PPR-CLW-00018",
            "urn:ngsi-ld:Pipe:PPR-CLW-00019",
            "urn:ngsi-ld:Pipe:PPR-CLW-00020",
            "urn:ngsi-ld:Pump:EQM-PUM-0111A",
            "urn:ngsi-ld:Pump:EQM-PUM-0111B",
            "urn:ngsi-ld:Pump:EQM-PUM-0111C",
            "urn:ngsi-ld:Pump:EQM-PUM-0111D",
            "urn:ngsi-ld:Pipe:PPR-CLW-00021",
            "urn:ngsi-ld:Pipe:PPR-CLW-00023",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101B"
          ]
        },
        "chilledWaterSupplyLineGroup": {
          "name": "냉수 공급관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00028",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPS-CDW-00016",
            "urn:ngsi-ld:Pipe:PPS-CDW-00017",
            "urn:ngsi-ld:Pipe:PPS-CDW-00015",
            "urn:ngsi-ld:Pipe:PPS-CDW-00024",
            "urn:ngsi-ld:Pipe:PPS-CDW-00012",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPS-CDW-00029",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00019",
            "urn:ngsi-ld:Pipe:PPS-CDW-00020",
            "urn:ngsi-ld:Pipe:PPS-CDW-00018",
            "urn:ngsi-ld:Pipe:PPS-CDW-00025",
            "urn:ngsi-ld:Pipe:PPS-CDW-00026",
            "urn:ngsi-ld:Pipe:PPS-CDW-00013",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPS-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPS-CDW-00022",
            "urn:ngsi-ld:Pipe:PPS-CDW-00023",
            "urn:ngsi-ld:Pipe:PPS-CDW-00021",
            "urn:ngsi-ld:Pipe:PPS-CDW-00027",
            "urn:ngsi-ld:Pipe:PPS-CDW-00014",
            "urn:ngsi-ld:Pipe:PPS-CDW-00011",
            "urn:ngsi-ld:Pump:EQM-PUM-0118A",
            "urn:ngsi-ld:Pump:EQM-PUM-0118B",
            "urn:ngsi-ld:Pump:EQM-PUM-0118C",
            "urn:ngsi-ld:Pipe:PPS-CDW-00008",
            "urn:ngsi-ld:Pipe:PPS-CDW-00009",
            "urn:ngsi-ld:Pipe:PPS-CDW-00010",
            "urn:ngsi-ld:Pipe:PPS-CDW-00006",
            "urn:ngsi-ld:Pipe:PPS-CDW-00005",
            "urn:ngsi-ld:Pipe:PPS-CDW-00002",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101B"
          ]
        },
        "chilledWaterReturnLineGroup": {
          "name": "냉수 환수관",
          "list": [
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00030",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151F",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151E",
            "urn:ngsi-ld:Pipe:PPR-CDW-00018",
            "urn:ngsi-ld:Pipe:PPR-CDW-00019",
            "urn:ngsi-ld:Pipe:PPR-CDW-00017",
            "urn:ngsi-ld:Pipe:PPR-CDW-00026",
            "urn:ngsi-ld:Pipe:PPR-CDW-00014",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-0112B",
            "urn:ngsi-ld:Pipe:PPR-CDW-00031",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151D",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151C",
            "urn:ngsi-ld:Pipe:PPR-CDW-00021",
            "urn:ngsi-ld:Pipe:PPR-CDW-00022",
            "urn:ngsi-ld:Pipe:PPR-CDW-00020",
            "urn:ngsi-ld:Pipe:PPR-CDW-00027",
            "urn:ngsi-ld:Pipe:PPR-CDW-00028",
            "urn:ngsi-ld:Pipe:PPR-CDW-00015",
            "urn:ngsi-ld:OutAirHandlingUnit:EQM-OAU-00113",
            "urn:ngsi-ld:Pipe:PPR-CDW-00032",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151B",
            "urn:ngsi-ld:AirHandlingUnit:EQM-AHU-0151A",
            "urn:ngsi-ld:Pipe:PPR-CDW-00024",
            "urn:ngsi-ld:Pipe:PPR-CDW-00025",
            "urn:ngsi-ld:Pipe:PPR-CDW-00023",
            "urn:ngsi-ld:Pipe:PPR-CDW-00029",
            "urn:ngsi-ld:Pipe:PPR-CDW-00016",
            "urn:ngsi-ld:Pipe:PPR-CDW-00001",
            "urn:ngsi-ld:Pipe:PPR-CDW-00002",
            "urn:ngsi-ld:Pipe:PPR-CDW-00003",
            "urn:ngsi-ld:Pump:EQM-PUM-0114A",
            "urn:ngsi-ld:Pump:EQM-PUM-0114B",
            "urn:ngsi-ld:Pump:EQM-PUM-0114C",
            "urn:ngsi-ld:Pump:EQM-PUM-0114D",
            "urn:ngsi-ld:Pipe:PPR-CDW-00004",
            "urn:ngsi-ld:Pipe:PPR-CDW-00005",
            "urn:ngsi-ld:Pipe:PPR-CDW-00006",
            "urn:ngsi-ld:Pipe:PPR-CDW-00007",
            "urn:ngsi-ld:Pipe:PPR-CDW-00009",
            "urn:ngsi-ld:Pipe:PPR-CDW-00012",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101B"
          ]
        },
        "gasSupplyLineGroup": {
          "name": "가스 공급관",
          "list": [
            "urn:ngsi-ld:Pipe:PPS-GAS-00001",
            "urn:ngsi-ld:Pipe:PPS-GAS-00004",
            "urn:ngsi-ld:DirectFiredAbsorptionChillerHeater:EQM-AHC-0101B"
          ]
        }
      }
    },
    "ratedCoolingWaterFlowRate": {
      "type": "Property",
      "value": "10180"
    },
    "@context": "[http://172.16.28.221:53005/e8ight-context.jsonld](http://172.16.28.221:53005/e8ight-context.jsonld)"
  }
]
```

## 내용
다음과 같이 월드상에 통신과 파싱을 맡는 액터를 제작했다.
우선 헤더.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "MyLab.h"  
#include "ML_ParserToCharacter.h"  
#include "GameFramework/Actor.h"  
#include "Interfaces/IHttpRequest.h"  
#include "ML_JsonParser.generated.h"  
  
class FHttpModule;  
  
UCLASS()  
class MYLAB_API AML_JsonParser : public AActor, public IML_ParserToCharacter  
{  
    GENERATED_BODY()  
      
public:   
    // Sets default values for this actor's properties  
    AML_JsonParser();  
  
protected:  
    // Called when the game starts or when spawned  
    virtual void BeginPlay() override;  
  
public:  
    UFUNCTION()  
    void HttpCall(const FString& _InURL, const FString& _InVerb);  
    void OnResponseReceived(FHttpRequestPtr _Request, FHttpResponsePtr _Response, bool _bWasSuccessful);  
  
    void ParseJson(TSharedPtr<FJsonObject> _JsonObject);  
      
private:  
    FHttpModule* Http;  
  
public:  
    UPROPERTY(EditAnywhere, BlueprintReadWrite)  
    FString URL;  
      
};
```

다음은 .cpp 파일.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "ML_JsonParser.h"  
  
namespace EQSDebug  
{  
    struct FItemData;  
}  
  
// Sets default values  
AML_JsonParser::AML_JsonParser()  
{  
    // Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.  
    PrimaryActorTick.bCanEverTick = false;  
  
    Http = &FHttpModule::Get();  
}  
  
// Called when the game starts or when spawned  
void AML_JsonParser::BeginPlay()  
{  
    Super::BeginPlay();  
  
    HttpCall(TEXT("http://52.79.98.84/entities"), TEXT("GET"));  
}  
  
void AML_JsonParser::HttpCall(FString _URL, FString _Type)  
{  
    TSharedRef<IHttpRequest> Request = Http->CreateRequest();  
    Request->OnProcessRequestComplete().BindUObject(this, &AML_JsonParser::OnResponseReceived);  
  
    Request->SetURL(_URL);  
    Request->SetVerb(_Type);  
    Request->SetHeader(TEXT("User-Agent"), "X-UnrealEngine-Agent");  
    Request->SetHeader("Content-Type", TEXT("application/json"));  
    Request->ProcessRequest();  
  
}  
  
void AML_JsonParser::OnResponseReceived(FHttpRequestPtr _Request, FHttpResponsePtr _Response, bool _bWasSuccessful)  
{  
    if (200 != _Response->GetResponseCode()) return;  
    const FString ResponseBody = _Response->GetContentAsString();  
    MLLOG(Warning, TEXT(" %s"), *ResponseBody)  
      
    TSharedPtr<FJsonValue> JsonValue;  
      
    const TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseBody);  
    if (FJsonSerializer::Deserialize(Reader, JsonValue))  
    {  
       MLLOG_S(Warning)  
    }  
} 

```

이 코드를 짜면서 주의한 점.
1. `TSharedPtr<FJsonObject>`가 반환타입에 오면 컴파일에러가 생기는 이슈가 있다(추후에 정확한 상황을 기술할 것임).
이는 `TSharedPtr<T>` 타입을 `UPROPERTY()`로 선언했거나 `OnResponseReceived(FHttpRequestPtr _Request, FHttpResponsePtr _Response, bool _bWasSuccessful)`를 `UFUNCTION`으로 선언했을 때 생기는 문제였음.

2. 그냥 `FString` 타입의 값들에 대해 `TEXT("")`를 붙여줄 것인지 고민하지 않아도 된다. 

3. 또, 관련 포스트들을 보면 `const TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(_Response->GetContentAsString());`의 템플릿 인수에 `TCHAR`를 넣는 사람들이 있는데, 반드시 넣어야 하는 건 아니다.
4. 이제 이걸 파싱하고 `TMap`에 담아야 한다. 근데 여기서 문제는, 다음과 같은 구조로 각 흡수식냉온수기에 관한 정보가 정리돼 있다는 것이다.
```JSON
[
	{...}
	{...}
	{...}
	{...}
]
```

즉, `lineGroup` 필드는 각 흡수식냉온수기별로 존재하는 배열이기에 단순히 `GetStringField(TEXT("lineGroup"))`으로 찾아올 수 없다는 것이다.
심지어 가장 상위의 배열은 필드 이름도 없다.
이런 이유로 기존의 `TSharedPtr<FJsonObject> FJsonObject`를 `Deserialize`에 넣는 방식은 그것의 반환타입을 `false`가 되게 했다.
이는 인수에 `TSharedPtr<FJsonValue> JsonValue`를 대입하는 것으로 해결했다.
![[Pasted image 20231110073138.png]]

이 중 하나의 흡수식 냉온수기에 속한 정적인 데이터만 모아서 표시하면 다음과 같다.
![[Pasted image 20231108211614.png]]

`MLLOG(Warning, TEXT(" _JsonString : %s"), *_Response->GetContentAsString());`의 로그를 통해 JSON을 받는 건 확인했다.
![[Pasted image 20231106214156.png]]

[[Unreal C++로 JSON 파일 불러와서 파싱 (2)]]에서 계속.