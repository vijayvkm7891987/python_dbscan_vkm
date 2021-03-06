# python_dbscan_vkm
Density_based_clustering
###for_base_map
!conda install -c conda-forge  basemap==1.1.0  matplotlib==2.2.2  -y 
###import_libraries
import pandas as pd
import numpy as np 
from sklearn.cluster import DBSCAN 
from sklearn.datasets.samples_generator import make_blobs 
from sklearn.preprocessing import StandardScaler 
import matplotlib.pyplot as plt 
%matplotlib inline
 
###data_creation
def createDataPoints(centroidLocation, numSamples, clusterDeviation):
    # Create random data and store in feature matrix X and response vector y.
    X, y = make_blobs(n_samples=numSamples, centers=centroidLocation, 
                                cluster_std=clusterDeviation)
    
    # Standardize features by removing the mean and scaling to unit variance
    X = StandardScaler().fit_transform(X)
    return X, y
	
###creating_data_points
X, y = createDataPoints([[4,3], [2,-1], [-1,4]] , 640, 0.5)

###modeling
epsilon = 0.3
minimumSamples = 7
db = DBSCAN(eps=epsilon, min_samples=minimumSamples).fit(X)
labels = db.labels_
labels

###distinguish_outliers
core_samples_mask = np.zeros_like(db.labels_, dtype=bool)
core_samples_mask[db.core_sample_indices_] = True
core_samples_mask

###no._of_clusters_ignore_noise
n_clusters_ = len(set(labels)) - (1 if -1 in labels else 0)
n_clusters_

####removing_repititions_in_labels_by_turing_into_set
unique_labels = set(labels)
unique_labels

###data_vis
colors = plt.cm.Spectral(np.linspace(0, 1, len(unique_labels)))

###plotting_with_colors
for k, col in zip(unique_labels, colors):
    if k == -1:
        # Black used for noise.
        col = 'k'

    class_member_mask = (labels == k)

    # Plot the datapoints that are clustered
    xy = X[class_member_mask & core_samples_mask]
    plt.scatter(xy[:, 0], xy[:, 1],s=50, c=[col], marker=u'o', alpha=0.5)

    # Plot the outliers
    xy = X[class_member_mask & ~core_samples_mask]
    plt.scatter(xy[:, 0], xy[:, 1],s=50, c=[col], marker=u'o', alpha=0.5)


###read_csv_file
data_vijay = pd.read_csv("c:/Users/vijay.mishra/Downloads/weather-stations20140101-20141231.csv")
data_vijay.head(5)

###cleaning_data
data_vijay = data_vijay[pd.notnull(data_vijay["Tm"])]
data_vijay = data_vijay.reset_index(drop=True)
data_vijay.head(5)

####data_visualization
from mpl_toolkits.basemap import Basemap
import matplotlib.pyplot as plt
from pylab import rcParams
%matplotlib inline
rcParams['figure.figsize'] = (14,10)

llon=-140
ulon=-50
llat=40
ulat=65

data_vijay = data_vijay[(data_vijay['Long'] > llon) & (data_vijay['Long'] < ulon) & (data_vijay['Lat'] > llat) &(data_vijay['Lat'] < ulat)]

my_map = Basemap(projection='merc',
            resolution = 'l', area_thresh = 1000.0,
            llcrnrlon=llon, llcrnrlat=llat, #min longitude (llcrnrlon) and latitude (llcrnrlat)
            urcrnrlon=ulon, urcrnrlat=ulat) #max longitude (urcrnrlon) and latitude (urcrnrlat)

my_map.drawcoastlines()
my_map.drawcountries()
# my_map.drawmapboundary()
my_map.fillcontinents(color = 'white', alpha = 0.3)
my_map.shadedrelief()

# To collect data based on stations        

xs,ys = my_map(np.asarray(data_vijay.Long), np.asarray(data_vijay.Lat))
data_vijay['xm']= xs.tolist()
data_vijay['ym'] =ys.tolist()

#Visualization1
for index,row in data_vijay.iterrows():
#   x,y = my_map(row.Long, row.Lat)
   my_map.plot(row.xm, row.ym,markerfacecolor =([1,0,0]),  marker='o', markersize= 5, alpha = 0.75)
#plt.text(x,y,stn)
plt.show()


###clustering of stations_based_on_lat_long

from sklearn.cluster import DBSCAN
import sklearn.utils
from sklearn.preprocessing import StandardScaler
sklearn.utils.check_random_state(1000)
Clus_dataSet = data_vijay[['xm','ym']]
Clus_dataSet = np.nan_to_num(Clus_dataSet)
Clus_dataSet = StandardScaler().fit_transform(Clus_dataSet)

# compute DBSCAN
db = DBSCAN(eps=0.15, min_samples=10).fit(Clus_dataSet)
core_samples_mask = np.zeros_like(db.labels_, dtype=bool)
core_samples_mask[db.core_sample_indices_] = True
labels = db.labels_
data_vijay["Clus_Db"]=labels

realClusterNum=len(set(labels)) - (1 if -1 in labels else 0)
clusterNum = len(set(labels)) 

# sample of clusters
data_vijay[["Stn_Name","Tx","Tm","Clus_Db"]].head(5)
