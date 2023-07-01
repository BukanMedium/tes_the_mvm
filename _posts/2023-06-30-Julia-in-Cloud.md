[ How To Run Julia in Cloud ]

Even tough julia was made to optimize hyper computation, often times the workload still surpass desktop computer capabilities. Once I tried running omniscape in a gaming pc with ryzen 9 5900x, and the computation process still taking 80 days. To overcome this, I tried cloud computing services like Julia Hub, GCP, AWS, etc. I think this is the right solution because I can access much more powerful machine easily (If you have fine budget, there's this 1TB RAM to up your game)

## in Julia Hub

The easiest way to run Julia in cloud is by using Julia Hub. In Julia Hub, Julia is already installed. So just start working as usual.

## in AWS or GCP
I tried this on Amazon and Google console with Linux. So it's probably just the same as if you want to install Julia in Linux. Here's the code:

'''
wget https://julialang-s3.julialang.org/bin/linux/x64/1.6/julia-1.8.5-linux-x86_64.tar.gz

tar zxvf julia-1.8.5-linux-x86_64.tar.gz

export PATH="$PATH:/home/shlhnj/julia-1.8.5/bin"

source ~/.bashrc

sudo mv julia-1.8.5/ /opt/

sudo ln -s /opt/julia-1.8.5/bin/julia /usr/local/bin/julia

julia

'''


To download and upload files from github:

'''
# Clone the GitHub repository
git clone https://github.com/BukanMedium/files.git

# Navigate to the directory containing the files you want to export
cd SON

# Copy the files to the local repository on your AWS shell instance
cp "cum_currmap.tif" /home/ec2-user/files



# Add the files to the staging area
git add cum_currmap_MAM.tif


# Commit the changes
git commit -m " Add cum_currmap_MAM.tif"



# Push the changes to the remote GitHub repository
git push origin main


'''
