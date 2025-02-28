### **Introduction Section Summary (Paragraph by Paragraph)**

1. **Photogrammetry Overview & Challenges**  
    Photogrammetry has **two major weaknesses**. 
	 - They require **hundreds of images** for accurate reconstruction
	 - The process is **fragmented**, using separate methods for **feature detection, Structure-from-Motion (SfM), and Multi-View Stereo (MVS)** e.g. keypoint detection algorithms are not trained with a 3D reconstruction loss. Lots of moving parts which at times not are engineered to be a perfect fit for each other.
    
2. **Existing Solutions & Limitations**  
    Some approaches use **diffusion models** to generate additional views from sparse inputs but finding Camera Poses for these low overlap sparse inputs is tough. **End-to-end optimizable models like PF-LRM and DUSt3R** attempt to jointly optimize pose estimation and scene reconstruction, thus eliminating the need for fragmented processes.
    
3. **Matrix3D Proposal**  
    Inspired by these methods, the authors introduce **Matrix3D**, a unified model capable of handling **pose estimation, depth estimation, and novel view synthesis**. It enables **sparse view reconstruction** using a **generative, multi-modal diffusion transformer (DiT)** that processes various modalities like images, camera poses, and depth maps.
    
4. **Key Innovations**  
    Matrix3D represents **all modalities in a unified 2D format**: **camera geometries as Plücker ray maps** and **3D structures as 2.5D depth maps**. This allows the model to **leverage generative image models** efficiently (just reuse their transformer architecture and weights after making few changes to channel number). Using **masked learning**, the model **predicts missing observations**, making it robust to **sparse and incomplete inputs** and allows substantial increase in volume of available training data by using bimodal image-pose or image-depth pairs.
    
5. **Final Reconstruction Process**  
     **densified camera and depth predictions**, can be used to initialize **3D Gaussian Splatting (3DGS)** for final reconstruction by back projecting the depth maps using . The method **eliminates the need for multiple task-specific models**, significantly improving efficiency.
 _Thinking_ : If predicted depth is related to actual depth through some affine relationship, can we guarantee their exist some camera parameters that can take this affine depth to be the actual depth and preserve similarity in geometry. 
$$
\begin{aligned}
{[X,Y,D]}^T= D . K^{-1}[x,y,1]^T  \\
{[aX+b,aY+b,aD+b]}^T=(aD +b).{K^*}^{-1}[x,y,1]^T \ \ \ \forall a\in R^+, b \ \ s.t. aD +b >0

\end{aligned}
$$
Can we prove such a {$K^*$} exists and if yes, would the structure would still be that of a valid intrinsic matrix? Let's assume 0 centered cameras and normalized camera coordinates.
$$
\begin{aligned}

x=(aD +b).{K^*}^{-1}X \\
aX+b= (aD + b)x/(f_x^*)\\
a(Dx/f_{x}) + b= (aD + b)x/(f_x^*)\\
f_x^* = f_x(aDx + bx)/(aDx +bf_x)

\end{aligned}
$$
This new focal length clearly depends on pixel value and hence, impossible. If we drop the similarity to original geometry then can be done MP.