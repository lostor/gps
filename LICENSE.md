
/**
 * 
 * @author hanm
 * <ul>
 * <li>地球坐标系 (WGS-84) 相互转火星坐标系 (GCJ-02) 的转换算法</li>
 * <ul>
 * <p>
 * </p>
 */
public class Gps {
    
    private static double pi = 3.14159265358979324D;// 圆周率
    
    private static double a = 6378245.0D;// WGS 长轴半径卫星椭球坐标投影到平面地图坐标系的投影因子。
    
    private static double ee = 0.00669342162296594323D;// WGS 偏心率的平方
    
    private static double x_pi = 3.14159265358979324 * 3000.0 / 180.0;
    
    class Point {
        
        private double longitude;
        
        private double latitude;
        
        Point(double lon, double lat) {
            this.longitude = lon;
            this.latitude = lat;
        }
        
        public double getLongitude() {
            return longitude;
        }
        
        public void setLongitude(double longitude) {
            this.longitude = longitude;
        }
        
        public double getLatitude() {
            return latitude;
        }
        
        public void setLatitude(double latitude) {
            this.latitude = latitude;
        }
        
    }
    
    /**
     * 中国坐标内
     * 
     * @param lat
     * @param lon
     * @return
     */
    public boolean outofChina(double lat, double lon) {
        if (lon < 72.004 || lon > 137.8347)
            return true;
        if (lat < 0.8293 || lat > 55.8271)
            return true;
        return false;
    }
    
    private double transformLat(double x, double y) {
        double ret = -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y + 0.1 * x * y + 0.2 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(y * pi) + 40.0 * Math.sin(y / 3.0 * pi)) * 2.0 / 3.0;
        ret += (160.0 * Math.sin(y / 12.0 * pi) + 320 * Math.sin(y * pi / 30.0)) * 2.0 / 3.0;
        return ret;
    }
    
    private double transformLon(double x, double y) {
        double ret = 300.0 + x + 2.0 * y + 0.1 * x * x + 0.1 * x * y + 0.1 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(x * pi) + 40.0 * Math.sin(x / 3.0 * pi)) * 2.0 / 3.0;
        ret += (150.0 * Math.sin(x / 12.0 * pi) + 300.0 * Math.sin(x / 30.0 * pi)) * 2.0 / 3.0;
        return ret;
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:34:14</li>
     * <li>两点之间的距离</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param latA
     * @param lonA
     * @param latB
     * @param lonB
     * @return
     */
    public double distance(double latA, double lonA, double latB, double lonB) {
        double earthR = 6371000.;
        double x = Math.cos(latA * pi / 180.) * Math.cos(latB * pi / 180.) * Math.cos((lonA - lonB) * pi / 180);
        double y = Math.sin(latA * pi / 180.) * Math.sin(latB * pi / 180.);
        double s = x + y;
        if (s > 1)
            s = 1;
        if (s < -1)
            s = -1;
        double alpha = Math.acos(s);
        double distance = alpha * earthR;
        return distance;
    }
    
    private Point delta(double lat, double lon) {
        double dLat = this.transformLat(lon - 105.0, lat - 35.0);
        double dLon = this.transformLon(lon - 105.0, lat - 35.0);
        double radLat = lat / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        return new Point(dLon, dLat);
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:35:28</li>
     * <li>墨卡托坐标转经纬度</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param mercatorLat
     * @param mercatorLon
     * @return
     */
    public Point mercatorDecrypt(double mercatorLat, double mercatorLon) {
        double x = mercatorLon / 20037508.34 * 180.;
        double y = mercatorLat / 20037508.34 * 180.;
        y = 180 / pi * (2 * Math.atan(Math.exp(y * pi / 180.)) - pi / 2);
        return new Point(x, y);
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:36:26</li>
     * <li>经纬度转墨卡托</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param wgsLat
     * @param wgsLon
     * @return
     */
    public Point mercatorEncrypt(double wgsLat, double wgsLon) {
        double x = wgsLon * 20037508.34 / 180.;
        double y = Math.log(Math.tan((90. + wgsLat) * pi / 360.)) / (pi / 180.);
        y = y * 20037508.34 / 180.;
        return new Point(x, y);
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:40:18</li>
     * <li>地球坐标系精确转火星坐标系</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param gcjLat
     * @param gcjLon
     * @return
     */
    public Point wgs84togcj02(double wgsLat, double wgsLon) {
        if (this.outofChina(wgsLat, wgsLon))
            return new Point(wgsLon, wgsLat);
        Point d = delta(wgsLat, wgsLon);
        return new Point(wgsLon + d.getLongitude(), wgsLat + d.getLatitude());
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:40:18</li>
     * <li>火星坐标系转地球坐标系</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param gcjLat
     * @param gcjLon
     * @return
     */
    public Point gcj02ToWgs84(double gcjLat, double gcjLon) {
        if (this.outofChina(gcjLat, gcjLon))
            return new Point(gcjLon, gcjLat);
        Point d = this.delta(gcjLat, gcjLon);
        return new Point(gcjLon - d.getLongitude(), gcjLat - d.getLatitude());
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:40:18</li>
     * <li>火星坐标系精确转地球坐标系</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param gcjLat
     * @param gcjLon
     * @return
     */
    public Point gcj02ToWgs84Exact(double gcjLat, double gcjLon) {
        double initDelta = 0.01;
        double threshold = 0.000000001;
        double dLat = initDelta, dLon = initDelta;
        double mLat = gcjLat - dLat, mLon = gcjLon - dLon;
        double pLat = gcjLat + dLat, pLon = gcjLon + dLon;
        double wgsLat, wgsLon, i = 0;
        while (true) {
            wgsLat = (mLat + pLat) / 2;
            wgsLon = (mLon + pLon) / 2;
            Point tmp = this.wgs84togcj02(wgsLat, wgsLon);
            dLat = tmp.getLatitude() - gcjLat;
            dLon = tmp.getLongitude() - gcjLon;
            if ((Math.abs(dLat) < threshold) && (Math.abs(dLon) < threshold))
                break;
            if (dLat > 0)
                pLat = wgsLat;
            else
                mLat = wgsLat;
            if (dLon > 0)
                pLon = wgsLon;
            else
                mLon = wgsLon;
            if (++i > 10000)
                break;
        }
        return new Point(wgsLon, wgsLat);
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:39:17</li>
     * <li>火星坐标系转百度坐标系</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param bdLat
     * @param bdLon
     * @return
     */
    public Point gcj02ToBd(double gcjLat, double gcjLon) {
        double x = gcjLon, y = gcjLat;
        double z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * x_pi);
        double theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * x_pi);
        double bdLon = z * Math.cos(theta) + 0.0065;
        double bdLat = z * Math.sin(theta) + 0.006;
        return new Point(bdLon, bdLat);
    }
    
    /**
     * 
     * <ul>
     * <li>author hanm</li>
     * <li>createDate: 2015年5月8日 下午4:39:17</li>
     * <li>百度坐标系转火星坐标系</li>
     * <p>
     * </p>
     * </ul>
     * 
     * @param bdLat
     * @param bdLon
     * @return
     */
    public Point bdToGcj02(double bdLat, double bdLon) {
        double x = bdLon - 0.0065, y = bdLat - 0.006;
        double z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * x_pi);
        double theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * x_pi);
        double gcjLon = z * Math.cos(theta);
        double gcjLat = z * Math.sin(theta);
        return new Point(gcjLon, gcjLat);
    }
    
}
