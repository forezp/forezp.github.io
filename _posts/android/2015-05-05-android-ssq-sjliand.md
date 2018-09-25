---
layout: post
title:  "省市区三级联动"
categories: android
tags:  android
---

* content
{:toc} 


最近项目，需要用到三级联动，在网上找了一些例子，进行了修改，实现，提炼出来了给大家分享
实现思路是在三个wheelview 进行联动。选择了省，马上就关联到市和区，选择了市 ，马上就可以关联到区。

<!--more-->

效果图：

<img src="http://img.blog.csdn.net/20160616165630865" width = "300"  alt="图片名称" align=center />
首先建了三个Model 用于存数据
存省 和市的list 和区的
```java
public class ProvinceInfoModel {
    private String name;
    private List<CityInfoModel> cityList;

    public ProvinceInfoModel() {
        super();
    }

    public ProvinceInfoModel(String name, List<CityInfoModel> cityList) {
        super();
        this.name = name;
        this.cityList = cityList;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<CityInfoModel> getCityList() {
        return cityList;
    }

    public void setCityList(List<CityInfoModel> cityList) {
        this.cityList = cityList;
    }

    @Override
    public String toString() {
        return "ProvinceInfoModel [name=" + name + ", cityList=" + cityList + "]";
    }
}


```
存市和其对应的区list
```java
public class CityInfoModel {

    private String name;
    private List<DistrictInfoModel> districtList;

    public CityInfoModel() {
        super();
    }

    public CityInfoModel(String name, List<DistrictInfoModel> districtList) {
        super();
        this.name = name;
        this.districtList = districtList;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<DistrictInfoModel> getDistrictList() {
        return districtList;
    }

    public void setDistrictList(List<DistrictInfoModel> districtList) {
        this.districtList = districtList;
    }

    @Override
    public String toString() {
        return "CityInfoModel [name=" + name + ", districtList=" + districtList
                + "]";
    }
}

```
区的modeL
```java
public class DistrictInfoModel {

    private String name;
    private String zipcode;

    public DistrictInfoModel() {
        super();
    }

    public DistrictInfoModel(String name, String zipcode) {
        super();
        this.name = name;
        this.zipcode = zipcode;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getZipcode() {
        return zipcode;
    }

    public void setZipcode(String zipcode) {
        this.zipcode = zipcode;
    }

    @Override
    public String toString() {
        return "DistrictInfoModel [name=" + name + ", zipcode=" + zipcode + "]";
    }

}
```
数据存储在xml中，在assets目录下，详情见源代码，代码太多了。

用的sax解析xml并得到数据存储在内存中
```java
public class AddrXmlParser extends DefaultHandler {

    private List<ProvinceInfoModel> provinceList = new ArrayList<ProvinceInfoModel>();


    public java.util.List<ProvinceInfoModel> getDataList() {
        return provinceList;
    }

    @Override
    public void startDocument() throws SAXException {

    }

    ProvinceInfoModel provinceModel = new ProvinceInfoModel();
    CityInfoModel cityModel = new CityInfoModel();
    DistrictInfoModel districtModel = new DistrictInfoModel();

    @Override
    public void startElement(String uri, String localName, String qName,
                             Attributes attributes) throws SAXException {
        if (qName.equals("province")) {
            provinceModel = new ProvinceInfoModel();
            provinceModel.setName(attributes.getValue(0));
            provinceModel.setCityList(new ArrayList<CityInfoModel>());
        } else if (qName.equals("city")) {
            cityModel = new CityInfoModel();
            cityModel.setName(attributes.getValue(0));
            cityModel.setDistrictList(new ArrayList<DistrictInfoModel>());
        } else if (qName.equals("district")) {
            districtModel = new DistrictInfoModel();
            districtModel.setName(attributes.getValue(0));
            districtModel.setZipcode(attributes.getValue(1));
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName)
            throws SAXException {
        if (qName.equals("district")) {
            cityModel.getDistrictList().add(districtModel);
        } else if (qName.equals("city")) {
            provinceModel.getCityList().add(cityModel);
        } else if (qName.equals("province")) {
            provinceList.add(provinceModel);
        }
    }

    @Override
    public void characters(char[] ch, int start, int length)
            throws SAXException {
    }
}

```

需要在activity 中开启线程读取数据

```java
protected boolean readAddrDatas() {
        List<ProvinceInfoModel> provinceList = null;
        AssetManager asset = getAssets();
        try {
            InputStream input = asset.open("province_data.xml");
            SAXParserFactory spf = SAXParserFactory.newInstance();
            SAXParser parser = spf.newSAXParser();
            AddrXmlParser handler = new AddrXmlParser();
            parser.parse(input, handler);
            input.close();
            provinceList = handler.getDataList();
            if (provinceList != null && !provinceList.isEmpty()) {
                mCurrentProviceName = provinceList.get(0).getName();
                List<CityInfoModel> cityList = provinceList.get(0).getCityList();
                if (cityList != null && !cityList.isEmpty()) {
                    mCurrentCityName = cityList.get(0).getName();
                    List<DistrictInfoModel> districtList = cityList.get(0).getDistrictList();
                    mCurrentDistrictName = districtList.get(0).getName();
                }
            }
            mProvinceDatas = new ArrayList<String>();
            for (int i = 0; i < provinceList.size(); i++) {
                mProvinceDatas.add(provinceList.get(i).getName());
                List<CityInfoModel> cityList = provinceList.get(i).getCityList();
                ArrayList<String> cityNames = new ArrayList<String>();
                for (int j = 0; j < cityList.size(); j++) {
                    cityNames.add(cityList.get(j).getName());
                    List<DistrictInfoModel> districtList = cityList.get(j).getDistrictList();
                    ArrayList<String> distrinctNameArray = new ArrayList<String>();
                    DistrictInfoModel[] distrinctArray = new DistrictInfoModel[districtList.size()];
                    for (int k = 0; k < districtList.size(); k++) {
                        DistrictInfoModel districtModel = new DistrictInfoModel(districtList.get(k).getName(), districtList.get(k).getZipcode());
                        distrinctArray[k] = districtModel;
                        distrinctNameArray.add(districtModel.getName());
                    }
                    mDistrictDatasMap.put(cityNames.get(j), distrinctNameArray);
                }
                mCitisDatasMap.put(provinceList.get(i).getName(), cityNames);
            }
            return true;
        } catch (Throwable e) {
            e.printStackTrace();
            return false;
        }
    }

```
读取完数据需要设置weelview 的数据

```java

 mProvincePicker.setOnSelectListener(new WheelView.OnSelectListener() {
            @Override
            public void endSelect(int id, String text) {
                String provinceText = mProvinceDatas.get(id);
                if (!mCurrentProviceName.equals(provinceText)) {
                    mCurrentProviceName = provinceText;
                    ArrayList<String> mCityData = mCitisDatasMap.get(mCurrentProviceName);
                    mCityPicker.resetData(mCityData);
                    mCityPicker.setDefault(0);
                    mCurrentCityName = mCityData.get(0);

                    ArrayList<String> mDistrictData = mDistrictDatasMap.get(mCurrentCityName);
                    mCountyPicker.resetData(mDistrictData);
                    mCountyPicker.setDefault(0);
                    mCurrentDistrictName = mDistrictData.get(0);
                }
            }

            @Override
            public void selecting(int id, String text) {
            }
        });
```

代码不一一写成，详情见源码。

[源码下载](http://download.csdn.net/detail/forezp/9551637)





