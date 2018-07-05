# Java-BMI-analysis-with-GUI
Java BMI analysis ,designed by GUI
> 1）将CollectionBMI类改造为GUIFileBMI类；
2）在GUIFileBMI类中增加方法saveFile(ArrayList&lt;Student> students, String filename)，该方法可以将学生信息students写入到指定的文本文件中，每一行写入一个学生。
3）在GUIFileBMI类中增加方法ArrayList&lt;Student>  readFile(String filename)读文件中的数据到学生students中。
4）在GUIFileBMI类中增加4个界面(Panel)：学生信息主界面、学生信息输入界面、学生信息维护界面、BMI统计界面；
a)	学生信息主界面为默认界面，该界面可从文件中读取并显示所有学生信息，并可以按不同属性对学生排序显示。该界面还可随机生成200名学生，并保存到文件中。
b)	学生信息输入界面每次输入单个学生，保存后，提示用户是否成功保存或提示用户已经存在，请重新输入(利用Dialog对话框进行提示)。
c)	学生信息维护界面要求输入要维护的学生学号，按学号查询并显示该学生所有信息，可删除或修改学生信息，并提示用户修改/删除成功或失败 (利用Dialog对话框进行提示)。
d)	BMI统计界面显示学生BMI的相关统计信息，并显示按BMI值划分的10个均等区间的学生人数的柱状图(利用JFreechart实现)。
5）在GUIFileBMI类中增加菜单，可通过菜单切换显示上述4各不同界面。

下面为源代码，但是菜单设计不够完善，无法实现菜单功能，只能是依次显示不同的JPanel来测试；另外删除学生信息的代码有问题。


```java
package oobmi;

import java.awt.GridLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Scanner;
import java.util.Collections;
import java.util.Comparator;
import java.io.*;
import java.util.Random;
import javax.swing.*;

import java.awt.Font;
import org.jfree.chart.ChartFactory;
import org.jfree.chart.ChartPanel;
import org.jfree.chart.JFreeChart;
import org.jfree.chart.axis.CategoryAxis;
import org.jfree.chart.axis.ValueAxis;
import org.jfree.chart.plot.CategoryPlot;
import org.jfree.chart.plot.PlotOrientation;
import org.jfree.data.category.CategoryDataset;
import org.jfree.data.category.DefaultCategoryDataset;

public class GUIFileBMI extends JFrame {

    public GUIFileBMI() {

	    JMenuBar menu = new JMenuBar();   //实例化菜单
        JMenu opMenu = new JMenu("Operations");//菜单的四个选项分别对应四个面板
        JMenuItem listitem1 = new JMenuItem("Main");
        listitem1.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                main();
            }
        }
        );
        JMenuItem listitem2 = new JMenuItem("Input");
        listitem2.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                input();
            }
        }
        );
        JMenuItem listitem3 = new JMenuItem("Change");
        listitem3.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                change();
            }
        }
        );
        JMenuItem listitem4 = new JMenuItem("Analysis");
        listitem4.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                analysis();
            }
        }
        );
        this.setBounds(600, 300, 600, 450);//设置程序窗口位置
        this.add(menu);
        menu.add(opMenu);
        opMenu.add(listitem1);
        opMenu.add(listitem2);
        opMenu.add(listitem3);
        opMenu.add(listitem4);
    }

    private void main() {   //选择主面板的函数
        this.setContentPane(new MainPanel());
        setVisible(true);
    }

    private void input() {   //选择输入面板的函数
        this.setContentPane(new InputPanel());
        setVisible(true);
    }

    private void change() {   //选择维护面板的函数
        this.setContentPane(new ChangePanel());
        setVisible(true);
    }

    private void analysis() {   //选择分析面板的函数
        this.setContentPane(new AnalysisPanel());
        setVisible(true);
    }
    int i, j, k = 0;                //定义循环变量
    ArrayList<Student> students = new ArrayList<Student>();

    public static void main(String[] args) {       //主函数
        GUIFileBMI gfb = new GUIFileBMI();       //实例化主类

        java.awt.EventQueue.invokeLater(new Runnable() {       //运行图形化界面
            public void run() {
                gfb.setVisible(true);
            }

        });
    }

    public void genStudents(int num, ArrayList<Student> students) {
        for (i = 0; i < num; i++) {
            Student s = new Student("", "", 1.7, 4, 2);
            students.add(s);
            s.id = "HIT1172800" + genRandomId(3);  //随机生成学号
            s.name = genRandomName(3);              //随机生成姓名
            s.height = Math.random() / 4 + 1.6;         //随机生成身高（1.60~1.85）
            s.weight = Math.random() * 30 + 50;       //随机生成体重（50~80）
            s.bmi = s.weight / s.height / s.height;
            BigDecimal h = new BigDecimal(s.height); //对身高保留两位小数
            s.height = h.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
            BigDecimal w = new BigDecimal(s.weight);  //对体重保留两位小数
            s.weight = w.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
            s.bmi = s.weight / s.height / s.height;
            BigDecimal bm = new BigDecimal(s.bmi);
            s.bmi = bm.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();//对bmi四舍五入保留两位小数
            if (!(isExists(s.id, students))) {
                i--;                                     //判断学号是否重复
            }
        }
    }

    public static String genRandomName(int length) {
        String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";//随机生成的姓名的每一位的选择
        Random random = new Random();                 //实例化random
        StringBuffer sb = new StringBuffer();         //实例化sb
        for (int j = 0; j < length; j++) {
            int number = random.nextInt(26);
            sb.append(str.charAt(number));
        }
        return sb.toString();
    }

    public static String genRandomId(int length) {
        String str = "1234567890";                   //随机生成的id后三位每一位的选择
        Random random = new Random();               //实例化random
        StringBuffer sb = new StringBuffer();           //实例化sb
        for (int j = 0; j < length; j++) {
            int number = random.nextInt(10);
            sb.append(str.charAt(number));
        }
        return sb.toString();
    }

    public void inputStudents() {
        Scanner in = new Scanner(System.in);
        System.out.println("请输入学生信息；输入结束后以空格回车完成输入。");
        String flag = "0";
        while (flag.compareTo(" ") != 0) {            //设定循环结束条件
            Student stu = new Student("", "", 0, 0, 0);
            stu.id = in.next();
            stu.name = in.next();
            stu.height = in.nextDouble();
            stu.weight = in.nextDouble();
            if (isExists(stu.id, students)) {
                System.out.println("学号重复，请重新输入:");
                continue;
            }
            students.add(stu);
            flag = in.nextLine();

        }
    }

    public boolean isExists(String id, ArrayList<Student> students) {
        for (Student s : students) {            //判断该ID是否出现过
            if (id.equals(s.id)) {
                return true;
            }
        }
        return false;
    }

    public double ave(ArrayList<Student> students) {
        double ave = 0;
        for (i = 0; i < students.size(); i++) {         //计算bmi总和
            ave += students.get(i).bmi;
        }
        ave /= students.size();                                     //计算平均数
        return ave;
    }

    public double med(ArrayList<Student> students) {
        int i, j;
        double temp;
        Student TempArray[] = students.toArray(new Student[students.size()]);
        for (i = 0; i < students.size(); i++) {                       //将bmi排序
            for (j = i + 1; j < students.size(); j++) {
                if (TempArray[j].bmi < TempArray[i].bmi) {
                    temp = TempArray[j].bmi;
                    TempArray[j].bmi = TempArray[i].bmi;
                    TempArray[i].bmi = temp;
                }
            }
        }
        if (students.size() % 2 == 0) {                         //就奇偶个学生分类
            return (TempArray[students.size() / 2].bmi + TempArray[students.size() / 2 - 1].bmi) / 2;
        } else {
            return TempArray[(students.size() - 1) / 2].bmi;
        }
    }

    public double[] mod(ArrayList<Student> students) {
        int times[] = new int[students.size()];         //定义每一个数值在其后出现的次数构成的数组
        double mode[] = new double[students.size()];//定义众数和0构成的数组
        int max = 0;                            //定义最大的次数
        for (i = 0; i < students.size(); i++) {
            for (j = i; j < students.size(); j++) {
                if (students.get(i).bmi == students.get(j).bmi) {
                    times[i]++;                 //对数组times[]赋值
                }
            }
        }
        max = times[0];
        for (i = 0; i < students.size(); i++) {            //使max的值为最大次数
            if (max <= times[i]) {
                max = times[i];
            }
        }
        for (i = 0; i < students.size(); i++) {
            if (max == times[i]) {
                mode[i] = students.get(i).bmi;      //对数组mode[]赋值
            }
        }
        return mode;                        //返回mod[]
    }

    public double var(ArrayList<Student> students) {
        double sum = 0;
        for (i = 0; i < students.size(); i++) {
            sum += students.get(i).bmi;
        }
        double ave = sum / students.size();             //计算平均值
        double var = 0;
        for (i = 0; i < students.size(); i++) {
            var += (students.get(i).bmi - ave) * (students.get(i).bmi - ave);       //方差的计算
        }
        var /= students.size();
        return var;                        //返回方差
    }

    private class IDComparator implements Comparator<Student> {//对ID排序

        @Override
        public int compare(Student st1, Student st2) {
            if (st1.id.compareTo(st2.id) < 0) {
                return 1;
            } else if (st1.id.compareTo(st2.id) > 0) {
                return -1;
            }
            return 0;
        }
    }

    private class NameComparator implements Comparator<Student> {   //对姓名排序

        @Override
        public int compare(Student st1, Student st2) {
            if (st1.name.compareTo(st2.name) < 0) {
                return 1;
            } else if (st1.name.compareTo(st2.name) > 0) {
                return -1;
            }
            return 0;
        }
    }

    private class HeightComparator implements Comparator<Student> { //对身高排序

        @Override
        public int compare(Student st1, Student st2) {
            if (st1.height < st2.height) {
                return 1;
            } else if (st1.height > st2.height) {
                return -1;
            }
            return 0;
        }
    }

    private class WeightComparator implements Comparator<Student> { //对体重排序

        @Override
        public int compare(Student st1, Student st2) {
            if (st1.weight < st2.weight) {
                return 1;
            } else if (st1.weight > st2.weight) {
                return -1;
            }
            return 0;
        }
    }

    private class BMIComparator implements Comparator<Student> {    //对BMI排序

        @Override
        public int compare(Student st1, Student st2) {
            if (st1.bmi < st2.bmi) {
                return 1;
            } else if (st1.bmi > st2.bmi) {
                return -1;
            }
            return 0;
        }
    }

    public void saveFile(ArrayList<Student> students, String filename) {       //保存文件
        try {
            BufferedWriter writer
                    = new BufferedWriter(new FileWriter(filename, false));
            for (Student st : students) {
                writer.write(String.format("%s,%s,%.2f,%.2f,%2f\r\n",
                        st.id, st.name, st.height, st.weight, st.bmi));
            }
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public ArrayList<Student> readFile(String filename) {       //读取文件
        File file = new File(filename);
        BufferedReader reader = null;
        ArrayList<Student> v = new ArrayList<Student>();
        try {
            reader = new BufferedReader(new FileReader(file));
            String tempString = null;
            while ((tempString = reader.readLine()) != null) {
                String[] a = tempString.split(",");
                Student st = new Student(a[0], a[1], Double.parseDouble(a[2]), Double.parseDouble(a[3]), Double.parseDouble(a[4]));
                v.add(st);
            }
            reader.close();
            return v;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return v;
    }

    public ArrayList<Student> SortByID(ArrayList<Student> students) {       //按照id排序
        GUIFileBMI bmi = new GUIFileBMI();
        Collections.sort(students, bmi.new IDComparator().reversed());
        return students;
    }

    public ArrayList<Student> SortByName(ArrayList<Student> students) {       //按照姓名排序
        GUIFileBMI bmi = new GUIFileBMI();
        Collections.sort(students, bmi.new NameComparator().reversed());
        return students;
    }

    public ArrayList<Student> SortByHeight(ArrayList<Student> students) {       //按照身高排序
        GUIFileBMI bmi = new GUIFileBMI();
        Collections.sort(students, bmi.new HeightComparator().reversed());
        return students;
    }

    public ArrayList<Student> SortByWeight(ArrayList<Student> students) {       //按照体重排序
        GUIFileBMI bmi = new GUIFileBMI();
        Collections.sort(students, bmi.new WeightComparator().reversed());
        return students;
    }

    public ArrayList<Student> SortByBMI(ArrayList<Student> students) {       //按照bmi排序
        GUIFileBMI bmi = new GUIFileBMI();
        Collections.sort(students, bmi.new BMIComparator().reversed());
        return students;
    }

    class MainPanel extends JPanel {

        public void set(JTextArea ta, ArrayList<Student> stu1) {        //生成学生信息表格的函数
            ta.setText("              ID\t\tName\tHeight\tWeight\t  BMI\t\n");
            for (Student st : stu1) {
                ta.append(st.toString());
            }
            ta.setEditable(false);
        }

        public MainPanel() {

            JTextArea ta = new JTextArea("", 20, 50);
            JScrollPane sp = new JScrollPane(ta);
            ArrayList<Student> stu1 = new ArrayList<Student>();
            JButton gen = new JButton("genStudents");
            gen.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {       //生成随机的学生信息
                    stu1.clear();                                              //stu1替换成随机学生信息
                    genStudents(200, stu1);
                    saveFile(students, "random.txt");
                    set(ta, stu1);
                }
            });
            JButton in = new JButton("inputStudents");
            in.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {           //生成输入的学生信息
                    stu1.clear();                                  //stu1置换成输入的学生信息
                    stu1.addAll(readFile("D:\\NetbeansProject\\OOBMI\\input.txt"));
                    set(ta, stu1);
                }
            });
            JButton si = new JButton("Sort by ID");
            si.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {       //按照ID排序
                    SortByID(stu1);
                    set(ta, stu1);
                }
            });
            JButton sn = new JButton("Sort by Name");
            sn.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {       //按照姓名排序
                    SortByName(stu1);
                    set(ta, stu1);
                }
            });
            JButton sh = new JButton("Sort by Height");       //按照身高排序
            sh.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    SortByHeight(stu1);
                    set(ta, stu1);
                }
            });
            JButton sw = new JButton("Sort by Weight");       //按照体重排序
            sw.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    SortByWeight(stu1);
                    set(ta, stu1);
                }
            });
            JButton sbmi = new JButton("Sort by BMI");       //按照bmi排序
            sbmi.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    SortByBMI(stu1);
                    set(ta, stu1);
                }
            });

            this.add(sp);
            this.add(si);
            this.add(sn);
            this.add(sh);
            this.add(sw);
            this.add(sbmi);
            this.add(gen);
            this.add(in);
        }
    }

    class InputPanel extends JPanel {

        public InputPanel() {

            this.setLayout(null);
            JLabel ii = new JLabel("学生ID：");
            ii.setBounds(200, 10, 100, 100);
            JTextField iid = new JTextField();
            iid.setBounds(300, 45, 100, 30);

            JLabel in = new JLabel("学生姓名：");
            in.setBounds(200, 60, 100, 100);
            JTextField iname = new JTextField();
            iname.setBounds(300, 95, 100, 30);

            JLabel ih = new JLabel("学生身高：");
            ih.setBounds(200, 110, 100, 100);
            JTextField iheight = new JTextField();
            iheight.setBounds(300, 145, 100, 30);

            JLabel iw = new JLabel("学生体重：");
            iw.setBounds(200, 160, 100, 100);
            JTextField iweight = new JTextField();
            iweight.setBounds(300, 195, 100, 30);

            JButton input = new JButton("录入");
            input.setBounds(250, 250, 80, 30);

            this.add(ii);
            this.add(iid);
            this.add(in);
            this.add(iname);
            this.add(ih);
            this.add(iheight);
            this.add(iw);
            this.add(iweight);
            this.add(input);

            ArrayList<Student> stu2 = new ArrayList<Student>();
            input.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    Student stu = new Student("", "", 0, 0, 0);                //临时的一个元素，用于保存输入的信息
                    stu.id = iid.getText();
                    stu.name = iname.getText();
                    stu.height = Double.parseDouble(iheight.getText());
                    stu.weight = Double.parseDouble(iweight.getText());
                    stu.bmi = stu.weight / stu.height / stu.height;
                    BigDecimal bm = new BigDecimal(stu.bmi);
                    stu.bmi = bm.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();       //计算bmi并四舍五入
                    if (isExists(stu.id, stu2)) {                          //提示输入失败，学号重复
                        JOptionPane.showMessageDialog(null,
                                "This id exists,please re-enter.", "Alert",
                                JOptionPane.ERROR_MESSAGE);
                    } else {                                                       //提示输入成功
                        JOptionPane.showMessageDialog(null,
                                "Successfully saved!", "Prompt",
                                JOptionPane.INFORMATION_MESSAGE);
                    }
                    if (!isExists(stu.id, stu2)) {
                        stu2.add(stu);                                 //添加输入的元素
                    }
                    iid.setText("");                                   //清空输入框
                    iname.setText("");
                    iheight.setText("");
                    iweight.setText("");
                    saveFile(stu2, "input.txt");               //保存文件
                }
            });
        }
    }

    class ChangePanel extends JPanel {

        public ChangePanel() {
            ArrayList<Student> stu3 = readFile("D:\\NetbeansProject\\OOBMI\\input.txt");       //创建数组读入学生信息
            this.setLayout(null);
            JLabel a = new JLabel("输入需要修改或删除的学生学号：");
            a.setBounds(20, 50, 200, 20);
            JTextField b = new JTextField();
            b.setBounds(240, 40, 100, 30);
            JTextArea c = new JTextArea("", 20, 50);
            c.setBounds(20, 90, 500, 50);
            c.setText("              ID\tName\tHeight\tWeight\t  BMI\t\n");
            JButton d = new JButton("OK");
            d.setBounds(440, 40, 100, 30);
            d.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {           //寻找指定id的学生并显示信息
                    for (Student s : stu3) {
                        if (s.id.compareTo(b.getText()) == 0) {
                            c.append(s.toString());
                        }
                    }
                }
            });

            JLabel f = new JLabel("输入修改后的信息：");
            f.setBounds(20, 170, 200, 15);
            JLabel g = new JLabel("姓名：");
            g.setBounds(20, 200, 100, 15);
            JLabel h = new JLabel("身高：");
            h.setBounds(20, 240, 100, 15);
            JLabel i = new JLabel("体重：");
            i.setBounds(20, 280, 100, 15);
            JTextField j = new JTextField();
            j.setBounds(100, 200, 100, 30);
            JTextField k = new JTextField();
            k.setBounds(100, 240, 100, 30);
            JTextField l = new JTextField();
            l.setBounds(100, 280, 100, 30);
            JButton m = new JButton("修改");
            m.setBounds(50, 330, 100, 30);
            m.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    for (Student s : stu3) {
                        if (s.id.compareTo(b.getText()) == 0) {       //将输入的信息保存到指定学生中
                            s.name = j.getText();
                            s.weight = Double.parseDouble(k.getText());
                            s.height = Double.parseDouble(l.getText());
                            s.bmi = s.weight / s.height / s.height;
                            BigDecimal bm = new BigDecimal(s.bmi);
                            s.bmi = bm.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
                            c.setText("              ID\t\tName\tHeight\tWeight\t  BMI\t\n");
                            c.append(s.toString());       //显示修改后的学生信息
                        }
                    }
                    JOptionPane.showMessageDialog(null,
                            "Successfully saved!", "Prompt",
                            JOptionPane.INFORMATION_MESSAGE);       //提示替换成功
                    saveFile(stu3, "input.txt");       //保存修改后的文件
                }
            });

            JButton n = new JButton("直接删除");
            n.setBounds(400, 240, 100, 30);
            n.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    for (Student s : stu3) {
                        if (s.id.compareTo(b.getText()) == 0) {
                            stu3.remove(s);       //直接删除指定学生信息
                        }
                    }
                    JOptionPane.showMessageDialog(null,
                            "Successfully deleted!", "Prompt",
                            JOptionPane.INFORMATION_MESSAGE);       //提示删除成功
                    saveFile(stu3, "input.txt");       //保存文件
                }
            });

            this.add(a);
            this.add(b);
            this.add(c);
            this.add(d);
            this.add(f);
            this.add(g);
            this.add(h);
            this.add(i);
            this.add(j);
            this.add(k);
            this.add(l);
            this.add(m);
            this.add(n);

        }
    }

    class AnalysisPanel extends JPanel {

        public AnalysisPanel() {
            this.setLayout(null);
            ArrayList<Student> stu4 = readFile("D:\\NetbeansProject\\OOBMI\\input.txt");
            JLabel a = new JLabel("人数：");
            a.setBounds(20, 40, 200, 15);
            JLabel b = new JLabel(String.valueOf(stu4.size()));       //显示人数
            b.setBounds(120, 40, 200, 15);
            JLabel c = new JLabel("BMI的平均数：");
            c.setBounds(20, 80, 100, 15);
            JLabel d = new JLabel(String.valueOf(ave(stu4)));       //显示平均数
            d.setBounds(120, 80, 200, 15);
            JLabel e = new JLabel("BMI的中位数：");
            e.setBounds(20, 120, 100, 15);
            JLabel f = new JLabel(String.valueOf(med(stu4)));       //显示中位数
            f.setBounds(120, 120, 200, 15);
            JLabel g = new JLabel("BMI的众数：");
            g.setBounds(20, 160, 100, 15);
            JLabel h = new JLabel();
            String m = "";
            for (double mod : mod(stu4)) {
                m = m + mod + " ";
            }                                                                      //显示众数
            h.setText(m);
            h.setBounds(120, 160, 200, 15);
            JLabel i = new JLabel("BMI的方差：");
            i.setBounds(20, 200, 100, 15);
            JLabel j = new JLabel(String.valueOf(var(stu4)));       //显示方差
            j.setBounds(120, 200, 200, 15);

            DefaultCategoryDataset data = new DefaultCategoryDataset();//创建数据集对象
            int v1 = 0, v2 = 0, v3 = 0, v4 = 0, v5 = 0, v6 = 0, v7 = 0, v8 = 0;       //定义每个区间的学生人数
            for (Student s : stu4) {
                if (17 <= s.bmi && s.bmi < 19) {
                    v1++;
                }
                if (19 <= s.bmi && s.bmi < 21) {
                    v2++;
                }
                if (21 <= s.bmi && s.bmi < 23) {
                    v3++;
                }
                if (23 <= s.bmi && s.bmi < 25) {
                    v4++;
                }
                if (25 <= s.bmi && s.bmi < 27) {
                    v5++;
                }
                if (27 <= s.bmi && s.bmi < 29) {
                    v6++;
                }
                if (29 <= s.bmi && s.bmi < 31) {
                    v7++;
                }
                if (31 <= s.bmi && s.bmi < 33) {
                    v8++;
                }
            }
            data.addValue(v1, "17~19", "17~19");       //将数据存储在数据集中
            data.addValue(v2, "19~21", "19~21");
            data.addValue(v3, "21~23", "21~23");
            data.addValue(v4, "23~25", "23~25");
            data.addValue(v5, "25~27", "25~27");
            data.addValue(v6, "27~29", "27~29");
            data.addValue(v7, "29~31", "29~31");
            data.addValue(v8, "31~33", "31~33");
            CategoryDataset dataset = data;
            //创建图形实体对象
            JFreeChart chart = ChartFactory.createBarChart3D(//工厂模式
                    "学生BMI分析", //图形的标题
                    "BMI", //目录轴的显示标签(X轴)
                    "学生数量", //数据轴的显示标签(Y轴)
                    dataset, //数据集
                    PlotOrientation.VERTICAL, //垂直显示图形
                    true, //是否生成图样
                    false, //是否生成提示工具
                    false);//是否生成URL链接
            CategoryPlot plot = chart.getCategoryPlot();//获取图形区域对象
            //------------------------------------------获取X轴
            CategoryAxis domainAxis = plot.getDomainAxis();
            domainAxis.setLabelFont(new Font("黑体", Font.BOLD, 14));//设置X轴的标题的字体
            domainAxis.setTickLabelFont(new Font("宋体", Font.BOLD, 12));//设置X轴坐标上的字体
            //-----------------------------------------获取Y轴
            ValueAxis rangeAxis = plot.getRangeAxis();
            rangeAxis.setLabelFont(new Font("黑体", Font.BOLD, 15));//设置Y轴坐标上的标题字体
            //设置图样的文字样式
            chart.getLegend().setItemFont(new Font("黑体", Font.BOLD, 15));
            //设置图形的标题
            chart.getTitle().setFont(new Font("宋体", Font.BOLD, 20));
            ChartPanel localChartPanel = new ChartPanel(chart, false);
            this.add(localChartPanel);
            localChartPanel.setBounds(400, 10, 600, 600);
            this.add(a);
            this.add(b);
            this.add(c);
            this.add(d);
            this.add(e);
            this.add(f);
            this.add(g);
            this.add(h);
            this.add(i);
            this.add(j);
        }
    }

    class Student {

        String id;              //定义学生id
        String name;        //定义学生姓名
        double height;      //定义学生身高
        double weight;      //定义学生体重
        double bmi;           //定义bmi

        public String toString() {
            return id + "\t" + name + "\t" + height + "\t" + weight + "\t" + bmi + "\n";
        }

        public Student(String id, String name, double height, double weight, double bmi) {//定义创建函数
            this.id = id;                                           //赋值
            this.name = name;
            this.weight = weight;
            this.height = height;
            this.bmi = bmi;

        }

    }
}

```

