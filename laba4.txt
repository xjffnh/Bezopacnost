import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.EOFException;
import java.io.FileInputStream;
import java.io.FileOutputStream;
 
public class TeaEcb {
    private final static int DELTA = 0x9e3779b9; 
    private final static int CYCLES = 32;
    private int sum;
    
    public static void main(String[] args){
        new TeaEcb();
    }
    
    TeaEcb() {
        String fname = "test.png";
        int key[] = {2, 112, 54, 8};
        try {
            encryptFile(fname, key);
            decryptFile(fname + "_encrypted", key);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public void encryptFile(String fname, int[] key) throws Exception {
        FileInputStream imgIn = new FileInputStream(fname);
        FileOutputStream imgOut = new FileOutputStream(fname + "_encrypted");
        DataInputStream dataIn = new DataInputStream(imgIn);
        DataOutputStream dataOut = new DataOutputStream(imgOut);
        int cipher[] = new int[2];
        boolean check = true;
        int[] img = new int[2]; 
        while(dataIn.available() > 0){
            try{
                img[0] = dataIn.readInt();
                check = true;
                img[1] = dataIn.readInt();
                cipher = encrypt(img, key); 
                dataOut.writeInt(cipher[0]);
                dataOut.writeInt(cipher[1]);
                check = false;
            }catch(EOFException e){  
                if(!check){                     
                    dataOut.writeInt(img[0]);
                    dataOut.writeInt(img[1]);
                }else                           
                    dataOut.writeInt(img[0]);
            }
        }
        dataIn.close();
        dataOut.close();
    }
    
    public void decryptFile(String fname, int[] key) throws Exception {
        FileInputStream imgIn = new FileInputStream(fname);
        FileOutputStream imgOut = new FileOutputStream(fname + "_decrypted");
        DataInputStream dataIn = new DataInputStream(imgIn);
        DataOutputStream dataOut = new DataOutputStream(imgOut);
        int plain[] = new int[2];
        int[] img = new int[2]; 
        boolean check = true;
        while(dataIn.available() > 0){
            try{                
                img[0] = dataIn.readInt();
                check = true;
                img[1] = dataIn.readInt();
                plain = decrypt(img, key);
                dataOut.writeInt(plain[0]);
                dataOut.writeInt(plain[1]);
                check = false;
            }catch(EOFException e){
                System.out.println("Cheeck > " + check);
                if(!check){
                    dataOut.writeInt(img[0]);
                    dataOut.writeInt(img[1]);
                }else
                    dataOut.writeInt(img[0]);;
            }
            
        }
        dataIn.close();
        dataOut.close();
    }
    
    private int[] encrypt(int[] data, int[] key) throws Exception{
        if(key == null){
            throw new Exception("Key is not defined!");
        }
        if (key.length != 4) {
            throw new Exception("Invalid key length!");
        }
        int left = data[0]; 
        int right = data[1];
        sum = 0;
        for(int i=0; i<CYCLES;i++){
            sum += DELTA;
            left += ((right << 4) + key[0]) ^ (right+sum) ^ ((right >> 5) + key[1]);
            right += ((left << 4) + key[2]) ^ (left+sum) ^ ((left >> 5) + key[3]);
 
        }
        int block[] = new int[2];
        block[0] = left;
        block[1] = right;
        return block;
    }
    
    private int[] decrypt(int[] data, int[] key) throws Exception{
        if(key == null){
            throw new Exception("Key is not defined!");
        }
        if (key.length != 4) {
            throw new Exception("Invalid key length!");
        }
        int left = data[0];
        int right = data[1];
        sum = DELTA << 5;
        for(int i=0; i<CYCLES;i++){
            right -= ((left << 4) + key[2]) ^ (left+sum) ^ ((left >> 5) + key[3]);
            left -= ((right << 4) + key[0]) ^ (right+sum) ^ ((right >> 5) + key[1]);
            sum -= DELTA;
        }
        int block[] = new int[2];
        block[0] = left;
        block[1] = right;
        return block;
    }
}